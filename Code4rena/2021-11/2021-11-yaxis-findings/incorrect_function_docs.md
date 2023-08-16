## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Incorrect function docs](https://github.com/code-423n4/2021-11-yaxis-findings/issues/35) 

# Handle

pmerkleplant


# Vulnerability details

The functions `AlToken::setBlacklist` and `AlToken::pauseAlchemist` in `v3/alchemix`
state in their docs: "This function reverts, if the caller does not have the admin role".

However, the functions revert if the caller does not have the **sentinel** role.

See [lines 93-98](https://github.com/code-423n4/2021-11-yaxis/blob/main/contracts/v3/alchemix/AlToken.sol#L93).

