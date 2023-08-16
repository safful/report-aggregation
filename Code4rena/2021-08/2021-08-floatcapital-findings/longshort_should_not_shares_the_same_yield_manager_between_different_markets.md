## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [LongShort should not shares the same Yield Manager between different markets](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/48) 

# Handle

jonah1005


# Vulnerability details

# LongShort should not shares the same Yield Manager between different markets
## Impact
The LongShort contract would not stop different markets from using the same yield manager contracts. Any extra aToken in the yield manager would be considered as market incentives in function `distributeYieldForTreasuryAndReturnMarketAllocation`. Thus, using the same yield manager for different markets would break the markets and allow users to withdraw fund that doesn't belong to them.


## Proof of Concept
https://github.com/code-423n4/2021-08-floatcapital/blob/main/contracts/contracts/YieldManagerAave.sol#L179-L204

## Tools Used
None

## Recommended Mitigation Steps
Given the fluency of programming skills the dev shows, I believe they wouldn't make this mistake on deployment. Still, I think there's space to improve in the YieldManagerAave contract. IMHO. As it's tightly coupled with longshort contract and its market logic, a initialize market function in the yield manager seems more reasonable.

