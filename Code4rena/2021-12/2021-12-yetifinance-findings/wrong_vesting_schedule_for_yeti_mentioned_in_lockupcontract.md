## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [Wrong vesting schedule for YETI mentioned in LockupContract](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/250) 

# Handle

kenzo


# Vulnerability details

LockupContract, LockupContractFactory amd ShortLockupContract all have comments that say:
```
Within the first year from deployment, the deployer of the YETIToken (Liquity AG's address) may transfer YETI only to valid LockupContracts, and no other addresses (this is enforced in YETIToken.sol's transfer() function).
The above two restrictions ensure that until one year after system deployment, YETI tokens originating from Liquity AG cannot enter circulating supply and cannot be staked to earn system revenue.
```

This comment is outdated (verified with sponsor). There is no such lockup on YETI tokens issued to team/treasury. (There might be other type of vesting which is probably implemented using TeamLockup.)

## Impact
Confusion, wrong description of team's capability to use yeti tokens issued.

## Proof of Concept
[Code ref](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/YETI/LockupContract.sol#L13:#L18).

## Recommended Mitigation Steps
Remove outdated comments.

