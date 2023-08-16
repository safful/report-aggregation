## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Making the MathLib internal](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/141) 

# Handle

UncleGrandpa925


# Vulnerability details

## Impact
Saving gas-cost for all transactions interacting with the pools. 

Currently the bytecode size of the Exchange is 10.99KB. Making the entire MathLib internal (therefore embedding it into the Exchange) will only make the bytecode size grows to 14.45KB, which is well below the limit of 24576 bytes. Doing this will save at least 2300 gas for every transaction since that the cost for cold-load the bytecode of the library, and also saving the gas cost of doing delegate call to the library instead of doing internal call.

## Recommended Mitigation Steps
Converting all public properties in the MathLib to internal.

## Note 
Normally I'm not into farming gas-optimization issues, but I think this is worth doing. 

