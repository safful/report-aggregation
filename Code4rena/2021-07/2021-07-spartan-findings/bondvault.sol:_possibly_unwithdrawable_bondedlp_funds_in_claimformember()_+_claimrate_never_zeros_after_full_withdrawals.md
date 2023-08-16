## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [BondVault.sol: Possibly unwithdrawable bondedLP funds in claimForMember() + claimRate never zeros after full withdrawals](https://github.com/code-423n4/2021-07-spartan-findings/issues/42) 

# Handle

hickuphh3


# Vulnerability details

### Impact

A host of problems arise from the L110-113 of the `claimForMember()` function, where `_claimable` is deducted from the bondedLP balance before the condition check, when it should be performed after (or the condition is changed to checking if the remaining bondedLP balance to zero).

```jsx
// L110 - L113
mapBondAsset_memberDetails[asset].bondedLP[member] -= _claimable; // Remove the claim amount from the user's remainder
if(_claimable == mapBondAsset_memberDetails[asset].bondedLP[member]){
	mapBondAsset_memberDetails[asset].claimRate[member] = 0; // If final claim; zero-out their claimRate
}
```

**1. Permanently Locked Funds**

If a user claims his bonded LP asset by calling `dao.claimForMember()`, or a malicious attacker helps a user to claim by calling `dao.claimAllForMember()`, either which is done such that `_claimable` is exactly half of his remaining bondedLP funds of an asset, then the other half would be permanently locked.

- Assume `mapBondAsset_memberDetails[asset].bondedLP[member] = 2 * _claimable`
- L110: `mapBondAsset_memberDetails[asset].bondedLP[member] = _claimable`
- L111: The if condition is satisfied
- L112: User's claimRate is erroneously set to 0 ⇒ `calcBondedLP()` will return 0, ie. funds are locked permanently

**2. Claim Rate Never Zeroes For Final Claim** 

On the flip side, should a user perform a claim that enables him to perform a full withdrawal (ie. `_claimable` = `mapBondAsset_memberDetails[asset].bondedLP[member]`, we see the following effects:

- L110: `mapBondAsset_memberDetails[asset].bondedLP[member] = 0`
- L111: The if condition is not satisfied, L112 does not execute, so the member's claimRate for the asset remains non-zero (it is expected to have been set to zero).

Thankfully, subsequent behaviour remains as expected since `calcBondedLP` returns zero as `claimAmount` is set to the member's bondedLP balance (which is zero after a full withdrawal).

### Recommended Mitigation Steps

The `_claimable` deduction should occur after the condition check. Alternatively, change the condition check to `if (mapBondAsset_memberDetails[asset].bondedLP[member] == 0)`.

