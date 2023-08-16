## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [It is possible to "uninitialize" `ERC20Facet` contract](https://github.com/code-423n4/2021-12-amun-findings/issues/276) 

# Handle

Czar102


# Vulnerability details

## Impact
The initialization status is defined by the [name and symbol](https://github.com/code-423n4/2021-12-amun/blob/main/contracts/basket/contracts/facets/ERC20/ERC20Facet.sol#L25-L28). It is possible it set them back to an empty string, uninitializing the contract and letting the `initialize(..)` function be called again. This way, the owner may, for example, hide minting additional tokens. Or, after accidentally setting name and symbol to empty strings, anyone can take control over the contract and mint any number of tokens.

In general, it shouldn't be possible to initialize more than once.

## Tools Used

Manual analysis

## Recommended Mitigation Steps

Consider adding empty string checks in `setName(...)` and `setSymbol(...)` functions.


