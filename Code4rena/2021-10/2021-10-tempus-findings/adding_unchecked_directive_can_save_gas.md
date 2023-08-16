## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2021-10-tempus-findings/issues/30) 

# Handle

WatchPug


# Vulnerability details

For the arithmetic operations that will never over/underflow, using the unchecked directive (Solidity v0.8 has default overflow/underflow checks) can save some gas from the unnecessary internal over/underflow checks.

For example:

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/TempusPool.sol#L375-L375

```solidity
uint256 timeToMaturity = (maturityTime > currentTime) ? (maturityTime - currentTime) : 0;
```

`maturityTime - currentTime` will never underflow.

