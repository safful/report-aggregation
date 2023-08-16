## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing sender address check in receive() may lead to locked Ether](https://github.com/code-423n4/2021-05-yield-findings/issues/48) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Add an address check in receive() of Ladle.sol to ensure the only address sending ETH being received in receive() is the Weth9 contract (similar to the check in PoolRouter.sol) for Ether withdrawal in _exitEther().

This will prevent stray Ether from being sent accidentally to this contract and getting locked.

## Proof of Concept

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L521-L522

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/yieldspace/PoolRouter.sol#L145-L148


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add an address check in receive() of Ladle.sol to ensure only Weth9 contract can send Ether to this contract.

