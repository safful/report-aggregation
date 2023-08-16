## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [BorrowerOperations has unused pieces of functionality](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/99) 

# Handle

hyh


# Vulnerability details

## Proof of Concept

_requireValidRouterParams and _requireRouterAVAXIndicesInOrder functions along with IYetiRouter interface are unused:

_requireValidRouterParams
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/BorrowerOperations.sol#L1053

_requireRouterAVAXIndicesInOrder
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/BorrowerOperations.sol#L1067

IYetiRouter import and the interface itself:
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/BorrowerOperations.sol#L12
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Interfaces/IYetiRouter.sol

## Recommended Mitigation Steps

If it is not meant to be implemented further consider removal to enhance code readability and size

