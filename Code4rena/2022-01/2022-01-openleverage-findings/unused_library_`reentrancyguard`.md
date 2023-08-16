## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unused library `ReentrancyGuard`](https://github.com/code-423n4/2022-01-openleverage-findings/issues/209) 

# Handle

WatchPug


# Vulnerability details

The library `ReentrancyGuard` is imported and inherited, but the modifier `nonReentrant` is unused.

https://github.com/code-423n4/2022-01-openleverage/blob/501e8f5c7ebaf1242572712626a77a3d65bdd3ad/openleverage-contracts/contracts/liquidity/LPoolDepositor.sol#L14-L14

```solidity
contract LPoolDepositor is ReentrancyGuard {
```

### Recommendation

Remove the import and change to:

```solidity
contract LPoolDepositor {
```

