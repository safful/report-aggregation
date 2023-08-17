## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [_revokeRole doesn't remove account from roleMember set](https://github.com/code-423n4/2022-04-backd-findings/issues/164) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/access/RoleManager.sol#L155


# Vulnerability details

## Impact
The function doesn't remove the address from _roleMembers[role] set, which will mess up with the roleCount

## Proof of Concept

## Tools Used

## Recommended Mitigation Steps
```
_roles[role].members[account] = false;
_roleMembers[role].remove(account);
```

