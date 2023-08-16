## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/261) 

# Handle

WatchPug


# Vulnerability details

For the arithmetic operations that will never over/underflow, using the unchecked directive (Solidity v0.8 has default overflow/underflow checks) can save some gas from the unnecessary internal over/underflow checks.

For example:

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/AssetWrappers/WJLP/ERC20_8.sol#L68-L70

```solidity=68
        require(_num_tokens <= balances[msg.sender], "You are trying to transfer more tokens than you have");

        balances[msg.sender] = balances[msg.sender] - _num_tokens;
```

`balances[msg.sender] - _num_tokens` will never underflow.

