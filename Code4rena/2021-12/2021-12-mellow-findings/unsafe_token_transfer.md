## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Unsafe token transfer](https://github.com/code-423n4/2021-12-mellow-findings/issues/88) 

# Handle

WatchPug


# Vulnerability details

Calling `ERC20.transfer()` without handling the returned value is unsafe.

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/ERC20Vault.sol#L81-L90

```solidity=81
function _pull(
        address to,
        uint256[] memory tokenAmounts,
        bytes memory
    ) internal override returns (uint256[] memory actualTokenAmounts) {
        for (uint256 i = 0; i < tokenAmounts.length; i++) {
            IERC20(_vaultTokens[i]).transfer(to, tokenAmounts[i]);
        }
        actualTokenAmounts = tokenAmounts;
    }
```

### Recommendation

Consider using OpenZeppelin's `SafeERC20` library with safe versions of transfer functions.

