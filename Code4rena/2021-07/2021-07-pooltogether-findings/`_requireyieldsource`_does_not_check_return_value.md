## Tags

- bug
- 1 (Low Risk)
- SwappableYieldSource
- sponsor confirmed

# [`_requireYieldSource` does not check return value](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/60) 

# Handle

cmichel


# Vulnerability details

The `_requireYieldSource` function performs a low-level status code and parses the return data even if the call failed as it does not check the first return value (`success`).
It could be the case that non-zero data is returned even though the call failed, and the function would return `true`.

Check the return value or perform a high-level call using the `_yieldSource` interface.

