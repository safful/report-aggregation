## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Loss Of Flash Governance Tokens If They Are Not Withdrawn Before The Next Request](https://github.com/code-423n4/2022-01-behodler-findings/issues/156) 

# Handle

kirk-baird


# Vulnerability details

## Impact

Users who have not called [withdrawGovernanceAsset()](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L142)  after  they have locked their tokens from a previous proposal (i.e. [assertGovernanceApproved](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L60)), will lose their tokens if [assertGovernanceApproved()](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L60) is called again with the same `target` and `sender`.

The `sender` will lose `pendingFlashDecision[target][sender].amount` tokens and the tokens will become unaccounted for and locked in the contract. Since the new amount is not added to the previous amount, instead the previous amount is overwritten with the new amount.

The impact of this is worsened by another vulnerability, that is [assertGovernanceApproved()](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L60) is a `public` function and may be called by any arbitrary user so long as the `sender` field has called `approve()` for `FlashGovernanceArbiter` on the ERC20 token. This would allow an attacker to make these tokens inaccessible for any arbitrary `sender`.

## Proof of Concept

In [assertGovernanceApproved()](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/FlashGovernanceArbiter.sol#L60) as seen below, the line`pendingFlashDecision[target][sender] = flashGovernanceConfig` will overwrite the previous contents. Thereby, making any previous rewards unaccounted for and inaccessible to anyone.

Note that we must wait `pendingFlashDecision[target][sender].unlockTime` between calls.

```
  function assertGovernanceApproved(
    address sender,
    address target,
    bool emergency
  ) public {
    if (
      IERC20(flashGovernanceConfig.asset).transferFrom(sender, address(this), flashGovernanceConfig.amount) &&
      pendingFlashDecision[target][sender].unlockTime < block.timestamp
    ) {
      require(
        emergency || (block.timestamp - security.lastFlashGovernanceAct > security.epochSize),
        "Limbo: flash governance disabled for rest of epoch"
      );
      pendingFlashDecision[target][sender] = flashGovernanceConfig;
      pendingFlashDecision[target][sender].unlockTime += block.timestamp;

      security.lastFlashGovernanceAct = block.timestamp;
      emit flashDecision(sender, flashGovernanceConfig.asset, flashGovernanceConfig.amount, target);
    } else {
      revert("LIMBO: governance decision rejected.");
    }
  }
```


## Recommended Mitigation Steps

Consider updating the initial if statement to ensure the `pendingFlashDecision` for that `target` and `sender` is empty, that is:
```
  function assertGovernanceApproved(
    address sender,
    address target,
    bool emergency
  ) public {
    if (
      IERC20(flashGovernanceConfig.asset).transferFrom(sender, address(this), flashGovernanceConfig.amount) &&
      pendingFlashDecision[target][sender].unlockTime == 0
    ) {
...
```

Note we cannot simply add the new `amount` to the previous `amount` incase the underlying `asset` has been changed.

