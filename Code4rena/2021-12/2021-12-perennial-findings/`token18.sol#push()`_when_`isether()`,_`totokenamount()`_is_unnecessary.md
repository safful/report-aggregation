## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`Token18.sol#push()` When `isEther()`, `toTokenAmount()` is unnecessary](https://github.com/code-423n4/2021-12-perennial-findings/issues/22) 

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

Therefore, in `Token18.sol#push()`, `toTokenAmount()` is unnecessary when `isEther()`.

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/utils/types/Token18.sol#L51-L59

```solidity=51
function push(
    Token18 self,
    address recipient,
    UFixed18 amount
) internal {
    isEther(self)
        ? Address.sendValue(payable(recipient), toTokenAmount(self, amount))
        : IERC20(Token18.unwrap(self)).safeTransfer(recipient, toTokenAmount(self, amount));
}
```

Can be changed to:

```solidity=51
function push(
    Token18 self,
    address recipient,
    UFixed18 amount
) internal {
    isEther(self)
        ? Address.sendValue(payable(recipient), UFixed18.unwrap(amount))
        : IERC20(Token18.unwrap(self)).safeTransfer(recipient, toTokenAmount(self, amount));
}
```

