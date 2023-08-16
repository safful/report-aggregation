## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Burning a User's Tokens for a Flash Proposal will not Deduct Their Balance](https://github.com/code-423n4/2022-01-behodler-findings/issues/157) 

# Handle

kirk-baird


# Vulnerability details

## Impact

The proposal to burn a user's tokens for a flash governance proposal does not result in the user losing any funds and may in fact unlock their funds sooner.

## Proof of Concept

The function [burnFlashGovernanceAsset()](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L124)  will simply overwrite the user's state with `pendingFlashDecision[targetContract][user] = flashGovernanceConfig;` as seen below.

```
  function burnFlashGovernanceAsset(
    address targetContract,
    address user,
    address asset,
    uint256 amount
  ) public virtual onlySuccessfulProposal {
    if (pendingFlashDecision[targetContract][user].assetBurnable) {
      Burnable(asset).burn(amount);
    }

    pendingFlashDecision[targetContract][user] = flashGovernanceConfig;
  }
```

Since `flashGovernanceConfig` is not modified in [BurnFlashStakeDeposit.execute()](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/Proposals/BurnFlashStakeDeposit.sol#L39) the user will have `amount` set to the current config amount which is likely what they originally transferred in {assertGovernanceApproved()](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L60). 

Furthermore, `unlockTime` will be set to the config unlock time.  The config unlock time is the length of time in seconds that proposal should lock tokens for not the future timestamp. That is unlock time may be say `7 days` rather than `now + 7 days`. As a result the check in [withdrawGovernanceAsset()](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L146)  `pendingFlashDecision[targetContract][msg.sender].unlockTime < block.timestamp,` will always pass unless there is a significant misconfiguration.

## Recommended Mitigation Steps

Consider deleting the user's data (i.e. `delete pendingFlashDecision[targetContract][user]`) rather than setting it to the config. This would ensure the user cannot withraw any funds afterwards.

Alternatively, only update `pendingFlashDecision[targetContract][user].amount` to subtract the amount sent as a function parameter and leave the remaining fields untouched.

