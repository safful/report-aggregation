## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Use of `matured` storage variable is unnecessary](https://github.com/code-423n4/2021-10-tempus-findings/issues/15) 

# Handle

TomFrench


# Vulnerability details

## Impact

Increased gas costs on several `TempusPool` functions

## Proof of Concept

`TempusPool`s have a `finalize` function which checks whether `block.timestamp >= maturityTime` and flips the `maturity` storage variable as well as setting `maturityInterestRate` to the current interest rate.

https://github.com/tempus-finance/tempus-protocol/blob/0240b4d172d7aa093a70e0401f4140c99aa30dc6/contracts/TempusPool.sol#L126-L135

`maturity` is used in several places to check whether the pool has expired however checking this variable is more expensive than checking `block.timestamp >= maturityTime` (due to the need for a SLOAD whereas `maturityTime` is immutable so no SLOAD is needed). I'd recommend making a `function matured() public view` to keep the readability.

However `maturityInterestRate` still needs to be set correctly. This could be done by reading the current interest rate when `matured()` returns true but `maturityInterestRate == 0` this could cause issues with some functions which are currently view functions however. 

## Recommended Mitigation Steps

Half solution:

Replace `matured` state variable with a `matured()` view function which returns `maturityInterestRate > 0`. This removes an SSTORE from `finalize` and an `SLOAD` from any function which uses `maturityInterestRate` as you can just check if it's greater than zero to see if the pool has matured.

Full solution:
 
Replace `matured` state variable with a `matured()` view function which returns  `block.timestamp >= maturityTime`. This combined with setting `maturityInterestRate` when you see that `matured() == true` and `maturityInterestRate == 0` would remove the need for the `finalize` function entirely.

