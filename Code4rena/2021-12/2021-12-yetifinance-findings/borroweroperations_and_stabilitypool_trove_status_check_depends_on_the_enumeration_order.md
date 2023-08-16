## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [BorrowerOperations and StabilityPool trove status check depends on the enumeration order](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/120) 

# Handle

hyh


# Vulnerability details

# Impact

Core system logic can break up if enumeration structure be updated.

## Proof of Concept

BorrowerOperations and StabilityPool check the active status of a trove by comparing TroveManager's getTroveStatus with 1:
BorrowerOperations:
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/BorrowerOperations.sol#L902
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/BorrowerOperations.sol#L907
StabilityPool:
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/StabilityPool.sol#L1104

TroveManagers inherit Status enumeration from TroveManagerBase:
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Dependencies/TroveManagerBase.sol#L72

## Recommended Mitigation Steps

With further system development it will be harder to track fixes needed on enumeration change.
Consider implementing TroveManager.isTroveActive(borrower) where trove.status is checked against Status.active and the corresponding boolean is returned.


