## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`getSharesForAmount` returns wrong value when `totalAssets == 0`](https://github.com/code-423n4/2022-03-prepo-findings/issues/55) 

# Lines of code

https://github.com/code-423n4/2022-03-prepo/blob/f63584133a0329781609e3f14c3004c1ca293e71/contracts/core/Collateral.sol#L328


# Vulnerability details

## Impact
The [`getSharesForAmount`](https://github.com/code-423n4/2022-03-prepo/blob/f63584133a0329781609e3f14c3004c1ca293e71/contracts/core/Collateral.sol#L328) function returns `0` if `totalAssets == 0`.

However, if **`totalSupply == 0`**, the actual shares that are minted in a [`deposit` are `_amount`](https://github.com/code-423n4/2022-03-prepo/blob/f63584133a0329781609e3f14c3004c1ca293e71/contracts/core/Collateral.sol#L83) even if `totalAssets == 0`.

Contracts / frontends that use this function to estimate their deposit when `totalSupply == 0` will return a wrong value.

## Recommended Mitigation Steps

```diff
function getSharesForAmount(uint256 _amount)
    external
    view
    override
    returns (uint256)
{
+   // to match the code in `deposit`
+   if (totalSupply() == 0) return _amount;

    uint256 _totalAssets = totalAssets();
    return
        (_totalAssets > 0)
            ? ((_amount * totalSupply()) / _totalAssets)
            : 0; // @audit this should be _amount according to `deposit`
}
```

