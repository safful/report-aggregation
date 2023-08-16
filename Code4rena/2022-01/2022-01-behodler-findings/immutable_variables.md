## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Immutable variables](https://github.com/code-423n4/2022-01-behodler-findings/issues/270) 

# Handle

pauliax


# Vulnerability details

## Impact
There are variables that do not change so they can be marked as immutable to greatly improve the gast costs. Examples of such variables are:
Limbo.sol
```solidity
  FlanLike Flan;
```
TokenProxyLike.sol
```solidity
  address internal baseToken;
```
ProposalFactory.sol
```solidity
  string public description;
  LimboDAOLike DAO;
```
Please review all the state variables and apply immutable where possible.

