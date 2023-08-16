## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Zero-address checks are missing](https://github.com/code-423n4/2021-10-union-findings/issues/3) 

# Handle

defsec


# Vulnerability details

## Impact

Zero-address checks are a best-practise for input validation of critical address parameters. While the codebase applies this to most addresses in setters, there are many places where this is missing in constructors and setters.

Impact: Accidental use of zero-addresses may result in exceptions, burn fees/tokens or force redeployment of contracts.

## Proof of Concept

The following code sections are missing zero address check.


https://github.com/code-423n4/2021-10-union/blob/main/contracts/treasury/Treasury.sol#L36
https://github.com/code-423n4/2021-10-union/blob/main/contracts/treasury/Treasury.sol#L104
https://github.com/code-423n4/2021-10-union/blob/main/contracts/treasury/TreasuryVester.sol#L19
https://github.com/code-423n4/2021-10-union/blob/main/contracts/token/Whitelistable.sol#L55
https://github.com/code-423n4/2021-10-union/blob/main/contracts/user/UserManager.sol#L157
https://github.com/code-423n4/2021-10-union/blob/main/contracts/user/UserManager.sol#L170
https://github.com/code-423n4/2021-10-union/blob/main/contracts/market/UToken.sol#L138
https://github.com/code-423n4/2021-10-union/blob/main/contracts/market/UToken.sol#L142
https://github.com/code-423n4/2021-10-union/blob/main/contracts/market/UToken.sol#L721
https://github.com/code-423n4/2021-10-union/blob/main/contracts/market/MarketRegistry.sol#L59
https://github.com/code-423n4/2021-10-union/blob/main/contracts/asset/AssetManager.sol#L78
https://github.com/code-423n4/2021-10-union/blob/main/contracts/asset/AssetManager.sol#L72
https://github.com/code-423n4/2021-10-union/blob/main/contracts/asset/AaveAdapter.sol#L89
https://github.com/code-423n4/2021-10-union/blob/main/contracts/asset/AaveAdapter.sol#L93


## Tools Used

None

## Recommended Mitigation Steps

Consider to Add zero-address checks.



