## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Constructor Does Not Check for Zero Addresses for _factory and _weth](https://github.com/code-423n4/2022-01-timeswap-findings/issues/104) 

# Handle

Meta0xNull


# Vulnerability details

## Impact
A wrong user input or wallets defaulting to the zero addresses for a missing input can lead to the contract needing to redeploy or wasted gas.

## Proof of Concept
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/TimeswapConvenience.sol#L62-L64

## Tools Used
Manual Review

## Recommended Mitigation Steps
requires Addresses is not zero.

require(_factory != address(0), "Address Can't Be Zero")
require(_weth != address(0), "Address Can't Be Zero")

