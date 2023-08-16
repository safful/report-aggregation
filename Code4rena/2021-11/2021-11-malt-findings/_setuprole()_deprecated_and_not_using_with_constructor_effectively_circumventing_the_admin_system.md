## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [_setupRole() Deprecated and Not Using With Constructor Effectively Circumventing the Admin System](https://github.com/code-423n4/2021-11-malt-findings/issues/115) 

# Handle

Meta0xNull


# Vulnerability details

## Impact
     * [WARNING]
     * ====
     * This function should only be called from the constructor when setting
     * up the initial roles for the system.
     *
     * Using this function in any other way is effectively circumventing the admin
     * system imposed by {AccessControl}.
     * ====
     *
     * NOTE: This function is deprecated in favor of {_grantRole}.

There are multiple contracts that import Permissions.sol and using Deprecated Function _setupRole() with Security Problem that Applicable to all these contracts because all of the contracts use initialize() Rather Than Constructor.


## Proof of Concept
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/AccessControl.sol#L174-L186
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Permissions.sol#L53
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Permissions.sol#L117
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Permissions.sol#L121

## Tools Used
Manual Review

## Recommended Mitigation Steps
Replace _setupRole() with _grantRole()

