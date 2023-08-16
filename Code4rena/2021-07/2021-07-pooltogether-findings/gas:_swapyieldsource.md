## Tags

- bug
- G (Gas Optimization)
- SwappableYieldSource
- sponsor confirmed

# [Gas: swapYieldSource](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/64) 

# Handle

cmichel


# Vulnerability details

The `SwappableYieldSource.swapYieldSource` function receives a `_newYieldSource` as a parameter and reads a `_currentYieldSource` from storage. A single storage read should therefore be enough for the entire function and sub-calls.

However, the `_transferFunds` function reads the new yield source from storage again, performing a second storage read. This can be optimized by `_transferFunds` taking an `oldYieldSource` and `newYieldSource` as parameters instead.


