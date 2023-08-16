## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing `_beforeTokenTransfer` Token Transfer Handle](https://github.com/code-423n4/2021-11-unlock-findings/issues/78) 

# Handle

hagrid


# Vulnerability details

## Details
`UnlockDiscountTokenV2.sol` has override for `_afterTokenTransfer` handler function to control events or operations after token transfers. Also, the `UnlockDiscountTokenV2.sol` uses another token contracts from `ERC20Patched.sol`. In `ERC20Patched.sol` contract there are also `_beforeTokenTransfer` transfer handles. However, the `UnlockDiscountTokenV2.sol`  token does not have any override for `_beforeTokenTransfer` method. 

## Impact
Contract will not react to any operations before token transfers. If gas calculations are aimed on UnlockToken's transfer, it will not be possible to calculate correct gas amounts without these both handlers (_beforeTokenTransfer and _afterTokenTransfer)

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

## Recommended Mitigation Steps
Possible fix is implementing additional override for `_beforeTokenTransfer` method:

```
 function _beforeTokenTransfer(address from, address to, uint256 amount) internal virtual override(ERC20Upgradeable, ERC20VotesUpgradeable) {
    return ERC20VotesUpgradeable._beforeTokenTransfer(from, to, amount);
  }
```

