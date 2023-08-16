## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Initialization parameters of new lock template are hardcoded](https://github.com/code-423n4/2021-11-unlock-findings/issues/137) 

# Handle

kenzo


# Vulnerability details

`setLockTemplate` is initializing the new template using hardcoded values.
This means that if a new lock version is set which has different/additional `initialize` parameters, Unlock protocol would have to be updated in order to initialize it.

## Impact
Less convenient adding of new locks as Unlock would have to be upgraded if their initialize function has changed.

## Proof of Concept
`setLockTemplate` uses the following code to initialize the template:
```
    IPublicLock(_publicLockAddress).initialize(
      address(this), 0, address(0), 0, 0, ''
    );
```
https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/Unlock.sol#L430:#L432
Which is hardcoded.
This is unlike `createLock` for example, where the initialize call is being received as parameter, to allow different future versions.
https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/Unlock.sol#L219


## Recommended Mitigation Steps
Change `setLockTemplate` so the initializing parameters would be received as parameter.

