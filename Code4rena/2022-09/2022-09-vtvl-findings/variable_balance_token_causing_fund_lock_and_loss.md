## Tags

- bug
- enhancement
- 2 (Med Risk)
- sponsor confirmed
- edited-by-warden

# [Variable balance token causing fund lock and loss](https://github.com/code-423n4/2022-09-vtvl-findings/issues/278) 

# Lines of code

https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L295
https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L388


# Vulnerability details

## Impact

Some ERC20 token's balance could change, one example is stETH. The balance could become insufficient at the time of `withdraw()`. User's fund will be locked due to DoS. The way to take the fund out is to send more token into the contract, causing fund loss to the protocol. And there is no guarantee that until the end time the balance would stay above the needed amount, the lock and loss issue persist.



## Proof of Concept

For stETH like tokens, the `balanceOf()` value might go up or down, even without transfer.
```solidity
// stETH
    function balanceOf(address who) external override view returns (uint256) {
        return _shareBalances[who].div(_sharesPerToken);
    }
```

In `VTVLVesting`, the `require` check for the spot `balanceOf()` value will pass, but it is possible that as time goes on, the value become smaller and fail the transfer. As a result, the `withdraw()` call will revert, causing DoS, and lock user's fund.
```solidity
// contracts/VTVLVesting.sol
    function _createClaimUnchecked() private  hasNoClaim(_recipient) {
        // ...
        require(tokenAddress.balanceOf(address(this)) >= numTokensReservedForVesting + allocatedAmount, "INSUFFICIENT_BALANCE");
        // ...
    }

    function withdraw() hasActiveClaim(_msgSender()) external {
        // ...
        tokenAddress.safeTransfer(_msgSender(), amountRemaining);
        // ...
    }
```


#### Reference
https://etherscan.io/address/0x312ca0592a39a5fa5c87bb4f1da7b77544a91b87#code


## Tools Used
Manual analysis.

## Recommended Mitigation Steps

Disallow such kind of variable balance token.