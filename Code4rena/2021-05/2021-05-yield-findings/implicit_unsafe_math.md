## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [Implicit unsafe math](https://github.com/code-423n4/2021-05-yield-findings/issues/24) 

# Handle

cmichel


# Vulnerability details

`Ladle._close` (and many other occurrences) reverts the transaction on certain signed inputs that are negated and cast to unsigned integers.

```solidity
// Ladle._close calling it with art or ink as type(int128).min will crash
uint128 amt = _debtInBase(vault.seriesId, series, uint128(-art));
ilkJoin.exit(to, uint128(-ink))

// explanation
int128 art = type(int128).min; // -2^127
uint128 amt = uint128(-art); // this fails as -art=--2^127=2^127 cannot be represented in int128
```

Other places:
- `CauldronMath.add`
- `Ladle._pour`
- everywhere where `-int*` is used

## Impact
One cannot use the actual `type(int128).min` value for function parameters.

## Recommended Mitigation Steps
Revert with a meaningful error message as is done in the `/math/Cast*` functions.


