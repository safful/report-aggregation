## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use the `unchecked` keyword to save gas](https://github.com/code-423n4/2021-07-connext-findings/issues/74) 

# Handle

shw


# Vulnerability details

## Impact

Using the `unchecked` keyword to avoid redundant arithmetic underflow/overflow checks to save gas when an underflow/overflow cannot happen.

## Proof of Concept

We can apply the `unchecked` keyword in the following lines of code since there are `require` statements before to ensure the arithmetic operations would not cause an integer underflow or overflow.

Referenced code:
[TransactionManager.sol#L125](https://github.com/code-423n4/2021-07-connext/blob/main/contracts/TransactionManager.sol#L125)
[TransactionManager.sol#L260](https://github.com/code-423n4/2021-07-connext/blob/main/contracts/TransactionManager.sol#L260)
[TransactionManager.sol#L364](https://github.com/code-423n4/2021-07-connext/blob/main/contracts/TransactionManager.sol#L364)
[TransactionManager.sol#L520](https://github.com/code-423n4/2021-07-connext/blob/main/contracts/TransactionManager.sol#L520)

## Recommended Mitigation Steps

For example, change the code at line 364 to:

```solidity
unchecked {
    uint256 toSend = txData.amount - relayerFee;
}
```

