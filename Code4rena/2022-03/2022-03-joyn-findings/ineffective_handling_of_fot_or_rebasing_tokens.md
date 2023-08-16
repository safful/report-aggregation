## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Ineffective Handling of FoT or Rebasing Tokens](https://github.com/code-423n4/2022-03-joyn-findings/issues/43) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/royalty-vault/contracts/RoyaltyVault.sol#L43-L50


# Vulnerability details

## Impact

Certain ERC20 tokens may change user's balances over time (positively or negatively) or charge a fee when a transfer is called (FoT tokens). The accounting of these tokens is not handled by `RoyaltyVault.sol` or `Splitter.sol` and may result in tokens being stuck in `Splitter` or overstating the balance of a user 

Thus, for FoT tokens if all users tried to claim from the Splitter there would be insufficient funds and the last user could not withdraw their tokens.

## Proof of Concept

The function `RoyaltyVault.sendToSplitter()` will transfer `splitterShare` tokens to the `Splitter` and then call `incrementWindow(splitterShare)` which tells the contract to split `splitterShare` between each of the users.

```
        require(
            IERC20(royaltyAsset).transfer(splitterProxy, splitterShare) == true,
            "Failed to transfer royalty Asset to splitter"
        );
        require(
            ISplitter(splitterProxy).incrementWindow(splitterShare) == true,
            "Failed to increment splitter window"
        );
```

Since the `Splitter` may receive less than `splitterShare` tokens if there is a fee on transfer the `Splitter` will overstate the amount split and each user can claim more than their value (except the last user who claims nothing as the contract will have insufficient funds to transfer them the full amount).

Furthermore, if the token rebase their value of the tokens down while they are sitting in the `Splitter` the same issue will occur. If the tokens rebase their value up then this will not be accounted for in the protocol.

## Recommended Mitigation Steps

It is recommend documenting clearly that rebasing token should not be used in the protocol.

Alternatively, if it is a requirement to handle rebasing tokens balance checks should be done before and after the transfer to ensure accurate accounting. Note: this makes the contract vulnerable to reentrancy and so a [reentrancy guard](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol) must be placed over the function `sendToSplitter()`.

```
        uint256 balanceBefore = IERC20(royaltyAsset).balanceOf(splitterProxy);
        require(
            IERC20(royaltyAsset).transfer(splitterProxy, splitterShare) == true,
            "Failed to transfer royalty Asset to splitter"
        );
        uint256 balanceAfter = IERC20(royaltyAsset).balanceOf(splitterProxy);
        require(
            ISplitter(splitterProxy).incrementWindow(balanceAfter - balanceBefore) == true,
            "Failed to increment splitter window"
        );
```

