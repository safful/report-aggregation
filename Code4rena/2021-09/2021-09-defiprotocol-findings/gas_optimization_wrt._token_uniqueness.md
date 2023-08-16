## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimization Wrt. Token Uniqueness](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/160) 

# Handle

leastwood


# Vulnerability details

## Impact

The `validateWeights()` function can be better optimised by using a hashmap to measure token uniqueness. Currently, the function utilises an `O(n^2)` solution. By first iterating through each hashmap index for `_tokens`, any previously set tokens can be first cleared . This improves the current solution to `O(n)`.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L53-L70

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider using a hashmap to measure token uniqueness. However, this hashmap needs to first be cleared out before using it each time in `validateWeights()`.

