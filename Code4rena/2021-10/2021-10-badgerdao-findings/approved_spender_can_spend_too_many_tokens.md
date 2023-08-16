## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Approved spender can spend too many tokens](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/43) 

# Handle

cmichel


# Vulnerability details

The `approve` function has not been overridden and therefore uses the internal _shares_, whereas `transfer(From)` uses the rebalanced amount.

## Impact
The approved spender may spend more tokens than desired. In fact, the approved amount that can be transferred keeps growing with `pricePerShare`.

Many contracts also use the same amount for the `approve` call as for the amount they want to have transferred in a subsequent `transferFrom` call, and in this case, they approve an amount that is too large (as the approved `shares` amount yields a higher rebalanced amount).

## Recommended Mitigation Steps
The `_allowances` field should track the rebalanced amounts such that the approval value does not grow. (This does not actually require overriding the `approve` function.)
In `transferFrom`, the approvals should then be subtracted by the _transferred_ `amount`, not the `amountInShares`:

```solidity
// _allowances are in rebalanced amounts such that they don't grow
// need to subtract the transferred amount
_approve(sender, _msgSender(), _allowances[sender][_msgSender()].sub(amount, "ERC20: transfer amount exceeds allowance"));
```

