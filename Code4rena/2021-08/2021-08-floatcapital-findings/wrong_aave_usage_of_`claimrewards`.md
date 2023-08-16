## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Wrong aave usage of `claimRewards`](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/49) 

# Handle

jonah1005


# Vulnerability details

## Impact
Aave yield manager claims rewards with the payment token. According to aave's document, aToken should be provided.
The aave rewards would be unclaimable.

## Proof of Concept
YieldManager's logic:
https://github.com/code-423n4/2021-08-floatcapital/blob/main/contracts/contracts/YieldManagerAave.sol#L161-L170

Reference:
https://docs.aave.com/developers/guides/liquidity-mining#claimrewards

## Tools Used
None
## Recommended Mitigation Steps
Change to
```solidity
    address[] memory rewardsDepositedAssets = new address[](1);
    rewardsDepositedAssets[0] = address(aToken);
```


