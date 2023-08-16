## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Can't revoke factory in NestedRecrods](https://github.com/code-423n4/2021-11-nested-findings/issues/203) 

# Handle

pauliax


# Vulnerability details

## Impact
NestedRecords contains no removeFactory function so there is no way to revoke a factory in case you no longer want to support it. This function is present in NestedAsset contract so I thought you might want to also have it here.

## Recommended Mitigation Steps
Consider if you are missing removeFactory or is this an intended functionality. 

