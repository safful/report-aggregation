## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`poolSizeLimit` does not account for differing unit values between borrow assets](https://github.com/code-423n4/2021-12-sublime-findings/issues/68) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

Inability to set a sensible range which covers all borrow assets without being overly wide.

## Proof of Concept

`PoolFactory` has a set requirement that created pool must ask to borrow an amount of assets which are within a certain range as encoded in the `poolSizeLimit`.

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/PoolFactory.sol#L286

This doesn't take into account the relative values of each borrow asset so it's hard to choose a one-size-fits-all value for these limits (A 100 WBTC loan could be sensible but a 100 USDC loan will be dwarfs by the gas costs to deploy the pool.)

## Recommended Mitigation Steps

Place limits on the USD value of the borrowed amount as reported by the factory's price oracle.

