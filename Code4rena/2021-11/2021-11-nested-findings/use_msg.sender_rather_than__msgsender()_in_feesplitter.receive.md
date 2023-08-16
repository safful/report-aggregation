## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [use msg.sender rather than _msgSender() in FeeSplitter.receive](https://github.com/code-423n4/2021-11-nested-findings/issues/1) 

# Handle

TomFrench


# Vulnerability details

## Impact

Gas costs

## Proof of Concept

In the `receive` function of `FeeSplitter` we check that the address sending ETH is the WETH contract:
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L74

As we can safely say that the WETH contract will never send a metatransaction, we can just use msg.sender and avoid the extra gas costs of `_msgSender()`

## Recommended Mitigation Steps

Replace `_msgSender()` with `msg.sender`

