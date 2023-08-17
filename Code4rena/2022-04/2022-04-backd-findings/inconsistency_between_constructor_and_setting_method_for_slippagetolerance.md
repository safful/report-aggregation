## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [Inconsistency between constructor and setting method for slippageTolerance](https://github.com/code-423n4/2022-04-backd-findings/issues/97) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/strategies/StrategySwapper.sol#L38-L43
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/strategies/StrategySwapper.sol#L109-L114


# Vulnerability details

## Impact
in the setSlippageTolerance(L119) method you have certain requirements to set slippageTolerance, but in the constructor you don't.



## Recommended Mitigation Steps
I would add the corresponding validations to the constructor


