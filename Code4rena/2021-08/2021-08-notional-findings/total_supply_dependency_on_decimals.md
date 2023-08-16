## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Total supply dependency on decimals](https://github.com/code-423n4/2021-08-notional-findings/issues/56) 

# Handle

pauliax


# Vulnerability details

## Impact
Total supply depends on the decimals. I think it makes sense to express totalSupply something like this to make it more readable and maintainable in case you will decide to change decimals:

    /// @notice EIP-20 token decimals for this token
    uint8 public constant decimals = 8;

    /// @notice Total number of tokens in circulation (100 million NOTE)
    uint256 public constant totalSupply = 10_000_0000 * 10 ** decimals;


