## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [DEPLOYER can drain DAOVault funds + manipulate proposal results](https://github.com/code-423n4/2021-07-spartan-findings/issues/27) 

# Handle

hickuphh3


# Vulnerability details

### Impact

2 conditions enable the `DEPLOYER` to drain the funds in the DAOVault.

- `DAOVault` is missing `purgeDeployer()` function
- `onlyDAO()` is callable by both the `DAO` and the `DEPLOYER`

The `DEPLOYER` can, at any time, call `depositLP()` to increase the LP funds of any account, then call `withdraw()` to withdraw the entire balance.

The only good use case for the `DEPLOYER` here is to help perform emergency withdrawals for users. However, this could use a separate modifier, like `onlyDeployer()`.

### Proof of Concept

1. `DEPLOYER` calls `depositLP()` with any arbitrary amount (maybe DAOVault's pool LP balance - Alice's deposited LP balance) for Alice and pool to increase their weight and balance.
2. At this point, Alice may vote for a proposal to swing it in her favour, or remove it otherwise (to implicitly vote against it)
3. `DEPLOYER` calls `withdraw()` for the Alice, which removes 100% of her balance (and therefore, the entire DAOVault's pool balance)

### Recommended Mitigation Steps

- Create a separate role and modifier for the `DEPLOYER`, so that he is only able to call `withdraw()` but not `depositLP()`
- Include the missing `purgeDeployer()` function.

