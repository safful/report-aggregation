## Tags

- bug
- G (Gas Optimization)
- sponsor acknowledged
- sponsor confirmed

# [Gas: Timestamp in router swap can be hardcoded](https://github.com/code-423n4/2021-09-yaxis-findings/issues/125) 

# Handle

cmichel


# Vulnerability details

When doing swaps with a Uniswap router from within a contract, there's no need to compute any offset from the current block for the `deadline` parameter.
The router just checks if `deadline >= block.timestamp`.

See `BaseStrategy._swapTokens` which does an unnecessary `block.timestamp` read and another unnecessary addition of `1800`.

## Recommended Mitigation Steps
The most efficient way to provide deadlines for a router swap is to use a hardcoded value that is far in the future, for example, `1e10`.


