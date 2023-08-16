## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`Vesting.sol#_claimableAmount()` Remove unnecessary storage variables can save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/246) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/vesting/contracts/Vesting.sol#L184-L184

```solidity=184
    benVested[_addr][1] = partial_sum;
```

`benVested[_addr][1]` is never used in the contract and the sum of partial claimable vesting is changing every second. Removing it can save gas.

