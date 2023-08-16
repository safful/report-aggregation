## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [FSDVesting: Optimise updateVestedTokens()](https://github.com/code-423n4/2021-11-fairside-findings/issues/31) 

# Handle

hickuphh3


# Vulnerability details

## Impact

1. The `_user` input in `updateVestedTokens()` is redundant because each user will have at most 1 vesting contract, and this function should be restricted to the FSD token contract only (kindly refer to related submitted issue), which stores and retrieves the mapping of users to vesting contracts.
2. The zero amount check is redundant because it is already checked in `FSD._createVesting()`.

## Recommended Mitigation Steps

```jsx
/**
* @dev Allows a vesting beneficiary to extend their vested token amount.
*/
function updateVestedTokens(uint256 _amount)
  external
  override
	onlyFsd
{
  amount = amount.add(_amount);
}
```

