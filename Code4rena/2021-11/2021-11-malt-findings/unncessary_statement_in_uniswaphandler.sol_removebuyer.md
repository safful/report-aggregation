## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unncessary statement in UniswapHandler.sol removeBuyer](https://github.com/code-423n4/2021-11-malt-findings/issues/377) 

# Handle

harleythedog


# Vulnerability details

## Impact
In UniswapHandler.sol within the `removeBuyer` function, there is a statement on line 308:
```
address buyer;
```

This variable is not used at all in the rest of the function, so this statement can be removed to save gas.

## Proof of Concept
See statement here: https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/DexHandlers/UniswapHandler.sol#L308

## Tools Used
Inspection

## Recommended Mitigation Steps
Remove unnecessary line to save gas

