## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Malicious token reward could disable withdrawals](https://github.com/code-423n4/2022-05-factorydao-findings/issues/20) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/e22a562c01c533b8765229387894cc0cb9bed116/contracts/PermissionlessBasicPoolFactory.sol#L230


# Vulnerability details

## Impact
`PermissionlessBasicPoolFactory.withdraw` requires each reward token transfers to succeed before withdrawing the deposit. If one of the reward token is a malicious/pausable contract that reverts on transfer, unaware users that deposited into this pool will have their funds stuck in the contract. 

## Recommended Mitigation Steps
Add an `emergencyWithdraw` function that ignores failed reward token transfers.

