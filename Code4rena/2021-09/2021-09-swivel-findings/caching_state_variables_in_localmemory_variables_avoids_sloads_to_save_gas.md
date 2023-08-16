## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Caching state variables in local/memory variables avoids SLOADs to save gas](https://github.com/code-423n4/2021-09-swivel-findings/issues/110) 

# Handle

0xRajeev


# Vulnerability details

## Impact

There are few places across contracts where the same state variables are read multiple times or the same external calls are made multiple times within a function. Caching state variables or results from external calls in local/memory variables avoids SLOADs and CALLs to save gas. Warm SLOADs cost 100 gas and CALLs cost 2600 gas after Berlin upgrade. MLOADs cost only 3 gas units.

## Proof of Concept

Cache swivel: https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L61-L64

Cache markets[u][m]:
https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L76-L86

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L99-L100

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L111-L112

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L181-L182

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L195-L196


Cache matured and maturityRate:
https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/vaulttracker/VaultTracker.sol#L156-L179\

Cache vaults:
https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/vaulttracker/VaultTracker.sol#L244


## Tools Used
Manual Analysis

## Recommended Mitigation Steps

Cache state variables or results from external calls in local/memory variables to save gas.

