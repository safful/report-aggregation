## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Keeper, not controller: Vault.setKeeper and utilize descriptions are incorrect](https://github.com/code-423n4/2022-01-insure-findings/issues/259) 

# Handle

hyh


# Vulnerability details



## Impact

`setKeeper` / `utilize` descriptions state that it is controller who is set / can run utilize, while keeper and controller are two separate roles, which don't have to coincide.

I.e. the descriptions now mix up the roles and are confusing this way.

## Proof of Concept

setKeeper:

https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Vault.sol#L499


utilize:

https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Vault.sol#L339


## Recommended Mitigation Steps

Update the descriptions to relate to the `keeper` role.


