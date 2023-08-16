## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Prevent the use of LOCK in setRoleAdmin to instead force the use of lockRole](https://github.com/code-423n4/2021-05-yield-findings/issues/50) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The LOCK role is special in AccessControl because it has itself as the admin role (like ROOT) but no members. This means that calling setRoleAdmin(msg.sig, LOCK) means no one can grant/revoke that msg.sig role anymore and it gets locked irreversibly. This means it disables admin-based permissioning management of that role and therefore is very powerful in its impact.

Given this, there is a special function lockRole() which is specifically meant to enforce LOCK as the admin for the specified role parameter. For all other role admin creations, the generic setRoleAdmin() may be used. However, setRoleAdmin() does not itself prevent specifying the use of LOCK as the admin. If this is accidentally used then it leads to disabling that role’s admin management irreversibly similar to the lockRole() function.

It is safer to force admins to use lockRole() as the only way to set admin to LOCK and prevent the use of LOCK as the adminRole parameter in setRoleAdmin(), because doing so will make the intention of the caller clearer as lockRole() clearly has that functionality specified in its name and that’s the only thing it does.


## Proof of Concept

Alice who is the admin for foo() wants to give the admin rights to Bob (0xFFFFFFF0) but instead of calling setRoleAdmin(foo.sig, 0xFFFFFFF0), she calls setRoleAdmin(foo.sig, 0xFFFFFFFF) where 0xFFFFFFFF is LOCK. This makes LOCK as the admin for foo() and prevents any further admin-based access control management for foo().

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/utils/access/AccessControl.sol#L48

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/utils/access/AccessControl.sol#L129-L131

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/utils/access/AccessControl.sol#L235-L240

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/utils/access/AccessControl.sol#L165-L176

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Prevent the use of LOCK as the adminRole parameter in setRoleAdmin().

