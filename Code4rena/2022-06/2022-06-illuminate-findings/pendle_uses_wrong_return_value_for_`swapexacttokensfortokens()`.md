## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Pendle Uses Wrong Return Value For `swapExactTokensForTokens()`](https://github.com/code-423n4/2022-06-illuminate-findings/issues/94) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L411


# Vulnerability details

## Impact

The function `swapExactTokensForTokens()` will return and array with the 0 index being the input amount follow by each output amount. The 0 index is incorrectly used in Pendle `lend()` function as the output amount. As a result the value of `returned` will be the invalid (i.e. the input rather than the output).

Since this impacts how many PTs will be minted to the `msg.sender`, the value will very likely be significantly over or under stated depending on the exchange rate. Hence the `msg.sender` will receive an invalid number of PT tokens.

## Proof of Concept

```solidity
            address[] memory path = new address[](2);
            path[0] = u;
            path[1] = principal;

            returned = IPendle(pendleAddr).swapExactTokensForTokens(a - fee, r, path, address(this), d)[0];
```


## Recommended Mitigation Steps

The amount of `principal` returned should be index 1 of the array returned by `swapExactTokensForTokens()`.

