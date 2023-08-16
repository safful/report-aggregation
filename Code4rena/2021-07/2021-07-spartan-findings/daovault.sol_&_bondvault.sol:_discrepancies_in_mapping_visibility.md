## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [DaoVault.sol & BondVault.sol: Discrepancies in mapping visibility](https://github.com/code-423n4/2021-07-spartan-findings/issues/53) 

# Handle

hickuphh3


# Vulnerability details

### Impact

In DaoVault and BondVault, the following mappings are declared private:

- `mapMember_weight`
- `mapMemberPool_weight`

The DaoVault has an additional private mapping `mapMemberPool_balance`.

Despite this, the DaoVault has getter methods for all 3 mappings, whilst the BondVault only has a getter method for `mapMember_weight`.

The getter methods (which aren't included in the interface) would be unnecessary if the mappings are declared as public. Also, the BondVault might perhaps be lacking a view method for `mapMemberPool_weight`. 

Should the separate getter methods remain unchanged, note that the getter method for `getMemberWeight()` has a convoluted implementation:

```jsx
function getMemberWeight(address member) external view returns (uint256) {
	if (mapMember_weight[member] > 0) {
		return mapMember_weight[member];
  } else {
    return 0;
    }
}
```

which can be simplified to simply returning the `mapMember_weight[member]`.

### Recommended Mitigation Steps

- Declare the relevant private mappings as public.
- Kindly check if `mapMemberPool_weight` should be public for the BondVault as well, since it is the case for the DaoVault.

