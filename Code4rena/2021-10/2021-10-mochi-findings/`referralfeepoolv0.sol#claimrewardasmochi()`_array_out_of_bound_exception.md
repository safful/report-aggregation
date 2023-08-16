## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`ReferralFeePoolV0.sol#claimRewardAsMochi()` Array out of bound exception](https://github.com/code-423n4/2021-10-mochi-findings/issues/97) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/feePool/ReferralFeePoolV0.sol#L28-L42

```solidity=28
function claimRewardAsMochi() external {
    IUSDM usdm = engine.usdm();
    address[] memory path = new address[](2);
    path[0] = address(usdm);
    path[1] = uniswapRouter.WETH();
    path[2] = address(engine.mochi());
    usdm.approve(address(uniswapRouter), reward[msg.sender]);
    // we are going to ingore the slippages here
    uniswapRouter.swapExactTokensForTokens(
        reward[msg.sender],
        1,
        path,
        address(this),
        type(uint256).max
    );
```

In `ReferralFeePoolV0.sol#claimRewardAsMochi()`, `path` is defined as an array of length 2 while it should be length 3.

As a result, at L33, an out-of-bound exception will be thrown and revert the transaction.

### Impact

`claimRewardAsMochi()` will not work as expected so that all the referral fees cannot be claimed but stuck in the contract.

