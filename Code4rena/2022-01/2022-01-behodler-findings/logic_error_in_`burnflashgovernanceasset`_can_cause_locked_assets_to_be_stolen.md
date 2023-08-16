## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Logic error in `burnFlashGovernanceAsset` can cause locked assets to be stolen](https://github.com/code-423n4/2022-01-behodler-findings/issues/305) 

# Handle

shw


# Vulnerability details

## Impact

A logic error in the `burnFlashGovernanceAsset` function that resets a user's `pendingFlashDecision` allows that user to steal other user's assets locked in future flash governance decisions. As a result, attackers can get their funds back even if they execute a malicious flash decision and the community burns their assets.

## Proof of Concept

1. An attacker Alice executes a malicious flash governance decision, and her assets are locked in the `FlashGovernanceArbiter` contract.
2. The community disagrees with Alice's flash governance decision and calls `burnFlashGovernanceAsset` to burn her locked assets. However, the `burnFlashGovernanceAsset` function resets Alice's `pendingFlashDecision` to the default config (see line 134).
3. A benign user, Bob executes another flash governance decision, and his assets are locked in the contract. 
4. Now, Alice calls `withdrawGovernanceAsset` to withdraw Bob's locked asset, effectively the same as stealing Bob's assets. Since Alice's `pendingFlashDecision` is reset to the default, the `unlockTime < block.timestamp` condition is fulfilled, and the withdrawal succeeds.

Referenced code:
[DAO/FlashGovernanceArbiter.sol#L134](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L134)
[DAO/FlashGovernanceArbiter.sol#L146](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L146)

## Recommended Mitigation Steps

Change line 134 to `delete pendingFlashDecision[targetContract][user]` instead of setting the `pendingFlashDecision` to the default.

