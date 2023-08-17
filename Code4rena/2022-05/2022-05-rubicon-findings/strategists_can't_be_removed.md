## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Strategists can't be removed](https://github.com/code-423n4/2022-05-rubicon-findings/issues/118) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/rubiconPools/BathHouse.sol#L264


# Vulnerability details

There is no option to revoke strategist's privilege.
As the strategist is a very strategic role which can effectively steal LP's funds, this is very dangerous.

## Impact
A rogue / compromised / cancelled strategist can not be revoked of permissions.

## Proof of Concept
There's a function to [approve](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/rubiconPools/BathHouse.sol#L264) a strategist, but no option to revoke the access.

## Recommended Mitigation Steps
Add a function / change the function and allow setting strategist's access to false.

