## Tags

- bug
- duplicate
- G (Gas Optimization)
- sponsor confirmed
- ERC20Rewards
- Strategy

# [Methods should be external instead of public](https://github.com/code-423n4/2021-08-yield-findings/issues/61) 

# Handle

hickuphh3


# Vulnerability details

### Impact

As brought up in a [previous audit issue](https://github.com/code-423n4/2021-05-yield-findings/issues/4), "the suggestion of changing all public auth functions to external auth will be applied". The same should therefore be done for the new contracts `Strategy.sol` and `ERC20Rewards.sol`, since all public methods in it aren't called internally.

