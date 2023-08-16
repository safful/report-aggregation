## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [FSDVesting: Claiming tributes should call FSD token's corresponding functions](https://github.com/code-423n4/2021-11-fairside-findings/issues/28) 

# Handle

hickuphh3


# Vulnerability details

## Impact

The claiming of staking and governance tributes for the a beneficiary's vested tokens should be no different than other users / EOAs. However, the `claimTribute()` and `claimGovernanceTribute()` are missing the actual claiming calls to the corresponding functions of the FSD token contract. As a result, the accrued rewards are taken from the beneficiary's vested token while not claiming (replenishing) from the FSD token contract.

## Recommended Mitigation Steps

In addition to what has been mentioned above, the internal accounting for claimedTribute states can be removed because they are already performed in the FSD token contract.

```jsx
// TODO: Remove _claimedTribute and _claimedGovernanceTribute mappings

/**
* @dev Allows claiming of staking tribute by `msg.sender` during their vesting period.
* It updates the claimed status of the vest against the tribute
* being claimed.
*
* Requirements:
* - claiming amount must not be 0.
*/
function claimTribute(uint256 num) external onlyBeneficiary {
    uint256 tribute = fsd.availableTribute(num);
    require(tribute != 0, "FSDVesting::claimTribute: No tribute to claim");
		fsd.claimTribute(num);
    fsd.safeTransfer(msg.sender, tribute);
    emit TributeClaimed(msg.sender, tribute);
}

/**
* @dev Allows claiming of governance tribute by `msg.sender` during their vesting period.
* It updates the claimed status of the vest against the tribute
* being claimed.
*
* Requirements:
* - claiming amount must not be 0.
*/
function claimGovernanceTribute(uint256 num) external onlyBeneficiary {
  uint256 tribute = fsd.availableGovernanceTribute(num);
  require(
    tribute != 0,
    "FSDVesting::claimGovernanceTribute: No governance tribute to claim"
  );
  fsd.claimGovernanceTribute(num);
  fsd.safeTransfer(msg.sender, tribute);
  emit GovernanceTributeClaimed(msg.sender, tribute);
}
```

