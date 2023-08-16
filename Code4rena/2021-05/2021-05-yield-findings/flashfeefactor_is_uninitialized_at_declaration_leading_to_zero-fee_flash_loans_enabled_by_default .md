## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [flashFeeFactor is uninitialized at declaration leading to zero-fee flash loans enabled by default ](https://github.com/code-423n4/2021-05-yield-findings/issues/53) 

# Handle

0xRajeev


# Vulnerability details

## Impact

flashFeeFactor is uninitialized at declaration and so zero initially until set by setFlashFeeFactor(). As indicated in one of the the explainer videos, the idea is to set this by default to uint256.max to disable flash loans by default.

Currently, flash loans are enabled by default with a zero flash fee unless changed by setFlashFeeFactor().

## Proof of Concept

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Join.sol#L26

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Join.sol#L32-L39

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Join.sol#L107-L110

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Join.sol#L117-L119

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Join.sol#L132


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Initialize at declaration with a reasonable value which could be uint256.max to disable flash loans by default.

