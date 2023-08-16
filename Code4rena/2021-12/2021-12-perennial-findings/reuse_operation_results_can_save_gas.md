## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Reuse operation results can save gas](https://github.com/code-423n4/2021-12-perennial-findings/issues/32) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/incentivizer/Incentivizer.sol#L232-L242

```solidity=232{234,238}
function claimFee(Token18[] calldata tokens) notPaused external {
    for(uint256 i; i < tokens.length; i++) {
        Token18 token = tokens[i];
        UFixed18 amount = fees[token];

        fees[token] = UFixed18Lib.ZERO;
        tokens[i].push(factory().treasury(), amount);

        emit FeeClaim(token, amount);
    }
}
```

`tokens[i]` at L238 is already cached in the local variable `token` at L234, resuing the result instead of doing the subscript operation again can save gas.

### Recommendation

Change

```solidity
tokens[i].push(factory().treasury(), amount);
```

to:

```solidity
token.push(factory().treasury(), amount);
```

