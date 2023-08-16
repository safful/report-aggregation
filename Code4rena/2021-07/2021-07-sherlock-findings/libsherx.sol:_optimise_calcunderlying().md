## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [LibSherX.sol: Optimise calcUnderlying()](https://github.com/code-423n4/2021-07-sherlock-findings/issues/38) 

# Handle

hickuphh3


# Vulnerability details

### Impact

- `tokens` can be directly be assigned to `gs.tokensSherX`.
- Redundant zero Initialization of the array element `amounts[i] = 0;` in the else case of `calcUnderlying()` (L69-L71).

### Recommended Mitigation Steps

```jsx
function calcUnderlying(uint256 _amount)
	external
	view
returns (IERC20[] memory tokens, uint256[] memory amounts)
{
	GovStorage.Base storage gs = GovStorage.gs();

  tokens = gs.tokensSherX;
  amounts = new uint256[](gs.tokensSherX.length);

  uint256 total = getTotalSherX();

  for (uint256 i; i < gs.tokensSherX.length; i++) {
    IERC20 token = tokens[i];
    if (total > 0) {
      PoolStorage.Base storage ps = PoolStorage.ps(token);
      amounts[i] = ps.sherXUnderlying.add(LibPool.getTotalAccruedDebt(token)).mul(_amount).div(
        total
      );
    }
  }
}
```

Gas reporter reports a gas reduction of ~150 gas. Gas savings should scale with number of underlying collaterals.

