## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Using 10**X for constants isn't gas efficient](https://github.com/code-423n4/2021-12-nftx-findings/issues/193) 

# Handle

Dravee


# Vulnerability details

## Impact
In Solidity, a `constant` expression in a variable will compute the expression everytime the variable is called. It's not the result of the expression that is stored, but the expression itself.

As Solidity supports the scientific notation, constants of form `10**X` can be rewritten as `1eX` to save the gas cost from the calculation with the exponentiation operator `**`.

## Proof of Concept
```
NFTXInventoryStaking.sol:
  28:     uint256 public constant BASE = 10**18;

NFTXMarketplaceZap.sol:
  158:   uint256 constant BASE = 10**18;

NFTXStakingZap.sol:
  163:   uint256 constant BASE = 10**18;

NFTXVaultUpgradeable.sol:
  33:     uint256 constant base = 10**18;
```

## Tools Used
Vs Code

## Recommended Mitigation Steps
Replace `10**18` with `1e18`

