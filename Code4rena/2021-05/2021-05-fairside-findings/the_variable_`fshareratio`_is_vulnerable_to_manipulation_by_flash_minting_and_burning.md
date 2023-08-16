## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity
- resolved

# [The variable `fShareRatio` is vulnerable to manipulation by flash minting and burning](https://github.com/code-423n4/2021-05-fairside-findings/issues/75) 

# Handle

shw


# Vulnerability details

## Impact

The variable `fShareRatio` in the function `purchaseMembership` of contract `FSDNetwork` is vulnerable to manipulation by flash minting and burning, which could affect several critical logics, such as the check of enough capital in the pool (line 139-142) and the staking rewards (line 179-182).

## Proof of Concept

The `fShareRatio` is calculated (line 136) by:

```solidity
(fsd.getReserveBalance() - totalOpenRequests).mul(1 ether) / fShare;
```

where `fsd.getReserveBalance()` can be significantly increased by a user minting a large amount of FSD tokens with flash loans. In that case, the increased `fShareRatio` could affect the function `purchaseMembership` results. For example, the user could purchase the membership even if the `fShareRatio` is < 100% previously, or the user could earn more staking rewards than before to reduce the membership fees. Although performing flash minting and burning might not be profitable overall since a 3.5% tribute fee is required when burning FSD tokens, it is still important to be aware of the possible manipulation of `fShareRatio`.

Referenced code:
[FSDNetwork.sol#L134-L142](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/network/FSDNetwork.sol#L134-L142)
[FSDNetwork.sol#L178-L182](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/network/FSDNetwork.sol#L178-L182)

## Recommended Mitigation Steps

Force users to wait for (at least) a block to prevent flash minting and burning.

