## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`rJoeAmount` can never be less than the `_avaxAmount`](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/201) 

# Handle

cmichel


# Vulnerability details

The `LaunchEvent.rJoePerAvax` variable is an _unscaled_ integer value and used to compute the `rJoeAmount` as:

```solidity
function getRJoeAmount(uint256 _avaxAmount) public view returns (uint256) {
    return _avaxAmount * rJoePerAvax;
}
```

This means the required `rJoeAmount` to burn can never be less than the deposited `avaxAmount`.
If a launch event desires to use `0.5 rJoe` per AVAX, this is not possible.

#### Recommendation
Consider the `rJoePerAvax` value as a value scaled by `1e18` and then divide by this scale in `getRJoeAmount` again.

