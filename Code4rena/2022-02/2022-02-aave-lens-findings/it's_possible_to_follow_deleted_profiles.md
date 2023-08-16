## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [It's possible to follow deleted profiles](https://github.com/code-423n4/2022-02-aave-lens-findings/issues/70) 

# Lines of code

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/libraries/InteractionLogic.sol#L49


# Vulnerability details

When someone tries to follow a profile, it checks if the handle exists, and if it doesn't, it reverts because the profile is deleted.
The problem is that there might be a new profile with the same handle as the deleted one, allowing following deleted profiles.


## Proof of Concept
Alice creates a profile with the handle "alice." The profile id is 1.
she deleted the profile.
she opens a new profile with the handle "alice". The new profile id is 2.
bob tries to follow the deleted profile (id is 1).
the check
```
if (_profileIdByHandleHash[keccak256(bytes(handle))] == 0)
	revert Errors.TokenDoesNotExist();
```
doesn't revert because there exists a profile with the handle "alice".
Therefore bob followed a deleted profile when he meant to follow the new profile.


## Recommended Mitigation Steps

change to:
```
if (_profileIdByHandleHash[keccak256(bytes(handle))] != profileIds[i])
	revert Errors.TokenDoesNotExist();
```


