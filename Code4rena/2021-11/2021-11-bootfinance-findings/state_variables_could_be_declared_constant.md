## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [State variables could be declared constant](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/88) 

# Handle

PranavG


# Vulnerability details

## Impact
State variables that never change can be declared constant. This can greatly reduce gas costs.
Examples :
airdrop_supply in AirdropDistribution.sol(#462)
investors_supply in InvestorDistribution.sol(#33) 
unixYear in Vesting.sol(#30)

## Recommended Mitigation Steps
Add the constant keyword for state variables whose value never change.

