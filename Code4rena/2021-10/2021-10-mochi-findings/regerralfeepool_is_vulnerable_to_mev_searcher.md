## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [regerralFeePool is vulnerable to MEV searcher](https://github.com/code-423n4/2021-10-mochi-findings/issues/62) 

# Handle

jonah1005


# Vulnerability details

# regerralFeePool is vulnerable to MEV searcher

## Impact
`claimRewardAsMochi` in the `ReferralFeePoolV0` ignores slippage. This is not a desirable design. There are a lot of MEV searchers in the current network. Swapping assets with no slippage control would get rekted. Please refer to https://github.com/flashbots/pm.

Given the current state of the Ethereum network. Users would likely be sandwiched. I consider this is a high-risk issue.

## Proof of Concept
[ReferralFeePoolV0.sol#L28-L48](https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/feePool/ReferralFeePoolV0.sol#L28-L48)

Please refer to https://medium.com/immunefi/mushrooms-finance-theft-of-yield-bug-fix-postmortem-16bd6961388f
 to see a possible attack pattern.


## Tools Used

None

## Recommended Mitigation Steps
I recommend adding minReceivedAmount as a parameter.

```solidity
function claimRewardAsMochi(uint256 _minReceivedAmount) external {
    // original logic here
    require(engine.mochi().balanceOf(address(this)) > _minReceivedAmount, "!min");
    engine.mochi().transfer(
        msg.sender,
        engine.mochi().balanceOf(address(this))
    );
}
```
Also, the front-end should calculate the min amount with the current price.

