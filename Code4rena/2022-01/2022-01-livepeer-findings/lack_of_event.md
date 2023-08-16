## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Lack of event](https://github.com/code-423n4/2022-01-livepeer-findings/issues/83) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Users and dapps are not notified when someting important is changed.

## Proof of Concept

Functions that are only executable by privileged users (e.g. onlyOwner) and have an impact (e.g. financial, trust) on other users should emit events.

- contracts\L1\gateway\L1LPTGateway.sol : [setCounterpart,setMinter].
- contracts\L2\gateway\L2Migrator.sol : [setL1Migrator,setDelegatorPoolImpl,setClaimStakeEnabled].
- contracts\L2\gateway\L2LPTGateway.sol : [setCounterpart].
- contracts\L2\gateway\L2LPTDataCache.sol : [setL1LPTDataCache,setL2LPTGateway].

## Tools Used
Manual review.

## Recommended Mitigation Steps
Emit event during important changes.

