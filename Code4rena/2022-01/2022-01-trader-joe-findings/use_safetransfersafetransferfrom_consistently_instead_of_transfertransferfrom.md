## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Use safeTransfer/safeTransferFrom consistently instead of transfer/transferFrom](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/12) 

# Handle

cccz


# Vulnerability details

## Impact

It is good to add a require() statement that checks the return value of token transfers or to use something like OpenZeppelin’s safeTransfer/safeTransferFrom unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

## Proof of Concept
https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L457

https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L463

https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L489

https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L513

https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L537

## Tools Used

Manual analysis

## Recommended Mitigation Steps

Consider using safeTransfer/safeTransferFrom or require() consistently.


