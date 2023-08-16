## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas optimization on InvestorDistribution.sol](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/94) 

# Handle

fr0zn


# Vulnerability details

## Vulnerability Details
The `InvestorDistribution` ([InvestorDistribution.sol](https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol)) contract does contain some statements that could be removed to optimize the gas usage.

## Impact
Gas optimization

## Tools Used
Manual code review

## Recommended Mitigation Steps
- Variable set on [Line 77](https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L77) can be removed since the implicit value is already zero
- On lines 106 and 107 ([here](https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L106-L107)), the statement could be changed to a single line containing `delete investors[_investor]`.

