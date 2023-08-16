## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [FYTokenFactory.sol - fyToken.ROOT() can be stored in a variable](https://github.com/code-423n4/2021-08-yield-findings/issues/10) 

# Handle

PierrickGT


# Vulnerability details

## Impact
In `FYTokenFactory.sol`, it is possible to avoid one sload by storing `fyToken.ROOT()` in a variable.

## Proof of Concept
https://github.com/code-423n4/2021-08-yield/blob/77bc292601ba6cb6d35f9a1cb606f21ed94ad36e/contracts/FYTokenFactory.sol#L37-L38

## Tools Used
Manual analysis

## Recommended Mitigation Steps
```
  bytes4 rootRole = fyToken.ROOT();

  fyToken.grantRole(rootRole, msg.sender);
  fyToken.renounceRole(rootRole, address(this));
```

