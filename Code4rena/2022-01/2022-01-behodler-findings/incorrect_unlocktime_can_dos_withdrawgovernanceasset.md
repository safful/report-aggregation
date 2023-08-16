## Tags

- bug
- duplicate
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Incorrect unlockTime can DOS withdrawGovernanceAsset](https://github.com/code-423n4/2022-01-behodler-findings/issues/228) 

# Handle

csanuragjain


# Vulnerability details

## Impact
unlockTime is set incorrectly

## Proof of Concept

1. Navigate to contract at https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol

2. Observe the assertGovernanceApproved function

```
function assertGovernanceApproved(
    address sender,
    address target,
    bool emergency
  ) public {
...
pendingFlashDecision[target][sender].unlockTime += block.timestamp;
...
}
```

3. Assume assertGovernanceApproved is called with sender x and target y and pendingFlashDecision[target][sender].unlockTime is 100 and block.timestamp is 10000 then

```
pendingFlashDecision[target][sender].unlockTime += block.timestamp; // 10000+100=10100
```

4. Again assertGovernanceApproved is called with same argument after timestamp 10100. This time unlockTime is set to very high value  (assume block.timestamp is 10500). This is incorrect

```
pendingFlashDecision[target][sender].unlockTime += block.timestamp; // 10100+10500=20600
```

## Recommended Mitigation Steps
unlock time should be calculated like below

```
constant public CONSTANT_UNLOCK_TIME = 1 days; // example
pendingFlashDecision[target][sender].unlockTime = CONSTANT_UNLOCK_TIME +  block.timestamp;
```

