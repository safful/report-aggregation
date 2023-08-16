## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`10 ** 18` can be changed to `1e18` and save some gas](https://github.com/code-423n4/2021-12-perennial-findings/issues/21) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/utils/types/Token18.sol#L151-L154

```solidity=151
function toTokenAmount(Token18 self, UFixed18 amount) private view returns (uint256) {
    UFixed18 conversion = UFixed18Lib.ratio(10 ** uint256(decimals(self)), 10 ** 18);
    return UFixed18.unwrap(amount.mul(conversion));
}
```

Can be changed to:

```solidity=151
function toTokenAmount(Token18 self, UFixed18 amount) private view returns (uint256) {
    UFixed18 conversion = UFixed18Lib.ratio(10 ** uint256(decimals(self)), 1e18);
    return UFixed18.unwrap(amount.mul(conversion));
}
```


https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/utils/types/Token18.sol#L163-L166

```solidity=163
function fromTokenAmount(Token18 self, uint256 amount) private view returns (UFixed18) {
    UFixed18 conversion = UFixed18Lib.ratio(10 ** 18, 10 ** uint256(decimals(self)));
    return UFixed18.wrap(amount).mul(conversion);
}
```

Can be changed to:

```solidity=163
function fromTokenAmount(Token18 self, uint256 amount) private view returns (UFixed18) {
    UFixed18 conversion = UFixed18Lib.ratio(1e18, 10 ** uint256(decimals(self)));
    return UFixed18.wrap(amount).mul(conversion);
}
```


