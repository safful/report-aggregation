## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unused Named Returns](https://github.com/code-423n4/2022-01-timeswap-findings/issues/81) 

# Handle

Dravee


# Vulnerability details

## Impact
Removing unused named return variables can reduce gas usage and improve code clarity.

## Proof of Concept
In `TimeswapPair:constantProduct()`, you return using both named return and actual return statement. To save gas and improve code quality consider using only one of those.

```
    /// @inheritdoc IPair
    function constantProduct(uint256 maturity)
        external
        view
        override
        returns (
            uint112 x,
            uint112 y,
            uint112 z
        )
    {
        State memory state = pools[maturity].state;
        return (state.x, state.y, state.z);
    }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Remove the unused named returns

