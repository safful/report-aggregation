## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Pool.sol: Optimizations](https://github.com/code-423n4/2021-07-spartan-findings/issues/49) 

# Handle

hickuphh3


# Vulnerability details

### Impact

1. `DEPLOYER` is set in the constructor but is not used anywhere in the contract
2. Redundant initialization `lastMonth = 0;`
3. `genesis` and `decimals` can have the `immutable` keywords since they are only set in the constructor and can't be changed
4. `iUTILS(_DAO().UTILS())` is called many times in `mintSynth()`, `removeForMember()` and `_swap*()` functions. Recommend storing as a local variable in these functions.
5. Since `revenueArray` cannot exceed length 2, the `addFee` function can be directly incorporated into the `addRevenue` function. Its for loop can be replaced with direct replacement of values. Also, `revenueArray.length != 2` is cleaner and easier to read compared to `!(revenueArray.length == 2)`. Given its purpose and usage, `archiveRevenue` / `cachePastRevenue` seems to be a better function name. If it is clear that revenueArray will be kept constant at 2, an alternative is to simply store the values as 2 separate variables.

### Recommended Mitigation Steps

1. Remove `DEPLOYER`
2. Remove the initialization `lastMonth = 0;`
3. `uint public immutable genesis;` and `uint8 public immutable override decimals;`
4. `iUTILS utils = _DAO().UTILS();` should utils be called more than once in a function
5. Possible implementation below

```jsx
function archiveRevenue(uint _totalRev) {
	if (revenueArray.length == 2) {
		// shift value to the right
		revenueArray[1] = revenueArray[0];
		revenueArray[0] = _totalRev;
  } else {
    // populate revenueArray to be of length 2
    revenueArray.push(_totalRev);
  }
}
```

