## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Wrong permissions on `reassignGlobalAdmin`](https://github.com/code-423n4/2021-11-malt-findings/issues/250) 

# Handle

cmichel


# Vulnerability details

The `Permissions.reassignGlobalAdmin` function is supposed to only be run with the `TIMELOCK_ROLE` role, see `onlyRole(TIMELOCK_ROLE, "Only timelock can assign roles")`.

However, the `TIMELOCK_ROLE` is not the admin of all the reassigned roles and the `revokeRole(role, oldAccount)` calls will fail as it requires the `ADMIN_ROLE`.

## Recommended Mitigation Steps
The idea might have been that only the `TIMELOCK` should be able to call this function, and usually it is also an admin, but the function strictly does not work if the caller _only_ has the `TIMELOCK` roll and will revert in this case.
Maybe governance decided to remove the admin role from the Timelock, which makes it impossible to call `reassignGlobalAdmin` anymore as both the timelock and admin are locked out.

