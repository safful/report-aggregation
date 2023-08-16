## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2022-01-yield-findings/issues/105) 

# Handle

WatchPug


# Vulnerability details

For the arithmetic operations that will never over/underflow, using the unchecked directive (Solidity v0.8 has default overflow/underflow checks) can save some gas from the unnecessary internal over/underflow checks.

For example:

1. https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexStakingWrapper.sol#L114-L114

```solidity
uint256 startIndex = rewardsLength - 1;
```

`rewardsLength - 1` will never underflow.

2. https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexYieldWrapper.sol#L82-L85

```solidity=82
bool isLast = i == vaultsLength - 1;
if (!isLast) {
    vaults_[i] = vaults_[vaultsLength - 1];
}
```

