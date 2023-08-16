## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Complex state variable copied to memory in redeemZcToken (MarketPlace.sol)](https://github.com/code-423n4/2021-09-swivel-findings/issues/41) 

# Handle

ye0lde


# Vulnerability details

## Impact
Operating on a copy of a state variable seems inefficient and confusing in this case.  
From a "gas" standpoint it's less efficient.  And future changes could render the addresses in the copied struct invalid if functions being called in redeemZcToken operate on the original state variable.

## Proof of Concept
The copy occurs here:
https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L123

The "mkt" variable is referenced here:
https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L131

## Tools Used
VS Code

## Recommended Mitigation Steps
Remove line #123:
https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L123

Replace "mkt" with "markets[u][m]" in line #131
https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L131


