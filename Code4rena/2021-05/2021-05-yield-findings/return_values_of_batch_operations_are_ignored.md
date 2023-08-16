## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Return values of batch operations are ignored](https://github.com/code-423n4/2021-05-yield-findings/issues/46) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Many batched operation functions return values but these are ignored by the caller batch(). While this may be acceptable for the front-end which picks up any state changes from such functions via emitted events, integrating protocols that make a call to batch() may require it to package and send back return values of all operations from the batch to react on-chain to the success/failure or other return values from such calls. Otherwise, they will be in the dark on the success/impact of batched operations they’ve triggered.

## Proof of Concept

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L120-L245

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L250

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L258

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L284-L286

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L296-L298

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L326-L328

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L342-L344

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L382-L384

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L396-L398

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L410-L412

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L446-L448

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L462-L464

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L527-L529

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L539-L541

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L559-L561

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L588-L590

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Package and send back return values of all batched operations’ functions to the caller of batch().

