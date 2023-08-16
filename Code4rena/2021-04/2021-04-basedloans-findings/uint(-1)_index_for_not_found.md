## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [uint(-1) index for not found](https://github.com/code-423n4/2021-04-basedloans-findings/issues/24) 

# Handle

paulius.eth


# Vulnerability details

## Impact
functions getTokenConfigBySymbolHash, getTokenConfigByCToken and getTokenConfigByUnderlying check returned index against max uint:
  index != uint(-1)
-1 should indicate that the index is not found, however, a default value for an uninitialized uint is 0, so it is impossible to get -1. What is even weirder is that 0 will be returned for non-existing configs but 0 is a valid index for the 1st config.

## Recommended Mitigation Steps
One of the solutions would be to reserve 0 for a not found index and use it when searching in mappings. Then normal indexes should start from 1. Another solution would be to introduce a new mapping with a boolean value that indicates if this index is initialized or not but this may be a more gas costly way.

