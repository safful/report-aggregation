## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`Token18.sol#balanceOf()` When `isEther()`, `fromTokenAmount()` is unnecessary](https://github.com/code-423n4/2021-12-perennial-findings/issues/23) 

# Handle

WatchPug


# Vulnerability details

When `isEther()`, `decimals` must be `18`:

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/utils/types/Token18.sol#L118-L120

```solidity=118
function decimals(Token18 self) internal view returns (uint8) {
    return isEther(self) ? 18 : IERC20Metadata(Token18.unwrap(self)).decimals();
}
```

Therefore, in `Token18.sol#balanceOf()`, `fromTokenAmount()` is unnecessary when `isEther()`.

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/utils/types/Token18.sol#L137-L142

```solidity=137
function balanceOf(Token18 self, address account) internal view returns (UFixed18) {
    uint256 tokenAmount = isEther(self) ?
        account.balance :
        IERC20(Token18.unwrap(self)).balanceOf(account);
    return fromTokenAmount(self, tokenAmount);
}
```

Can be changed to:

```solidity=137
function balanceOf(Token18 self, address account) internal view returns (UFixed18) {
    return isEther(self) ?
        UFixed18.wrap(account.balance) :
        fromTokenAmount(self, IERC20(Token18.unwrap(self)).balanceOf(account));
}
```

