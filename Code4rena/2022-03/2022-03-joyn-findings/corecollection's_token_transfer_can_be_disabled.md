## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [CoreCollection's token transfer can be disabled](https://github.com/code-423n4/2022-03-joyn-findings/issues/37) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/main/royalty-vault/contracts/RoyaltyVault.sol#L51-L57
https://github.com/code-423n4/2022-03-joyn/blob/main/splits/contracts/Splitter.sol#L164


# Vulnerability details

## Impact

When royaltyAsset is an ERC20 that doesn't allow zero amount transfers, the following griefing attack is possible, entirely disabling CoreCollection token transfer by precision degradation as both reward distribution and vault balance can be manipulated.

Suppose splitterProxy is set, all addresses and fees are configured correctly, system is in normal operating state.

POC:

Bob the attacker setup a bot which every time it observes positive royaltyVault balance:

1) runs `sendToSplitter()`, distributing the whole current royaltyAsset balance of the vault to splitter and platform, so vault balance becomes zero

2) sends `1 wei` of royaltyAsset to the royaltyVault balance

3) each next CoreCollection token transfer will calculate `platformShare = (balanceOfVault * platformFee) / 10000`, which will be 0 as platformFee is supposed to be less than 100%, and then there will be an attempt to transfer it to `platformFeeRecipient`

If royaltyAsset reverts on zero amount transfers, the whole operation will fail as the success of `IERC20(royaltyAsset).transfer(platformFeeRecipient, platformShare)` is required for each CoreCollection token transfer, which invokes `sendToSplitter()` in `_beforeTokenTransfer()` as vault balance is positive in (3).

Notice, that Bob needn't to front run the transfer, it is enough to empty the balance in a lazy way, so cumulative gas cost of the attack can be kept moderate.

Setting severity to medium as on one hand, the attack is easy to setup and completely blocks token transfers, making the system inoperable, and it looks like system has to be redeployed on such type of attack with some manual management of user funds, which means additional operational costs and reputational damage. On the another, it is limited to the zero amount reverting royaltyAsset case or the case when platformFee is set to 100%.

That is, as an another corner case, if platformFee is set to 100%, `platformShare` will be `1 wei` and `splitterShare` be zero in (3), so this attack be valid for any royaltyAsset as it is required in Splitter's `incrementWindow` that `splitterShare` be positive.

## Proof of Concept

As royaltyAsset can be an arbitrary ERC20 it can be reverting on zero value transfers:

https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers

`_beforeTokenTransfer` runs `IRoyaltyVault(royaltyVault).sendToSplitter()` whenever royaltyVault is set and have positive balance:

https://github.com/code-423n4/2022-03-joyn/blob/main/core-contracts/contracts/CoreCollection.sol#L307

`sendToSplitter()` leaves vault balance as exactly zero as `splitterShare = balanceOfVault - platformShare`, i.e. no dust is left behind:

https://github.com/code-423n4/2022-03-joyn/blob/main/royalty-vault/contracts/RoyaltyVault.sol#L41

This way the balance opens up for the tiny amount manipulation.

One require that can fail the whole operation is `platformShare` transfer:

https://github.com/code-423n4/2022-03-joyn/blob/main/royalty-vault/contracts/RoyaltyVault.sol#L51-L57

Another is positive `royaltyAmount` = `splitterShare` requirement:

https://github.com/code-423n4/2022-03-joyn/blob/main/splits/contracts/Splitter.sol#L164

## Recommended Mitigation Steps

The issue is that token transfer, which is core system operation, require fee splitting to be done on the spot. More failsafe design is to try to send the fees and record the amounts not yet distributed, not requiring immediate success. The logic here is that transfer itself is more important than fee distribution, which is simple enough and can be performed in a variety of ways later.

Another issue is a combination of direct balance usage and the lack of access controls of the sendToSplitter function, but it only affects fee splitting and is somewhat harder to address.

As one approach consider trying, but not requiring `IRoyaltyVault(royaltyVault).sendToSplitter()` to run successfully as it can be executed later with the same result.

Another, a simpler one (the same is in `Griefing attack is possible making Splitter's claimForAllWindows inaccessible` issue), is to introduce action threshold, `MIN_ROYALTY_AMOUNT`, to `sendToSplitter()`, for example:

Now:
```
/**
 * @dev Send accumulated royalty to splitter.
 */
function sendToSplitter() external override {
    uint256 balanceOfVault = getVaultBalance();

    require(
        balanceOfVault > 0,
        "Vault does not have enough royalty Asset to send"
    );
	...

    emit RoyaltySentToSplitter(...);
    emit FeeSentToPlatform(...);
}
```

To be:
```
/**
 * @dev Send accumulated royalty to splitter if it's above MIN_ROYALTY_AMOUNT threshold.
 */
function sendToSplitter() external override {
    uint256 balanceOfVault = getVaultBalance();

    if (balanceOfVault > MIN_ROYALTY_AMOUNT) {
		...

	    emit RoyaltySentToSplitter(...);
	    emit FeeSentToPlatform(...);
    }
}
```

