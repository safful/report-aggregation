## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`ConvexStakingWrapper.sol#` Switching between 1, 2 instead of 0, 1 is more gas efficient](https://github.com/code-423n4/2022-01-yield-findings/issues/102) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexStakingWrapper.sol#L54-L55

```solidity
    bool private constant _NOT_ENTERED = false;
    bool private constant _ENTERED = true;
```

https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexStakingWrapper.sol#L81-L90

```solidity
    modifier nonReentrant() {
        // On the first call to nonReentrant, _notEntered will be true
        require(_status != _ENTERED, "ReentrancyGuard: reentrant call");
        // Any calls to nonReentrant after this point will fail
        _status = _ENTERED;
        _;
        // By storing the original value once again, a refund is triggered (see
        // https://eips.ethereum.org/EIPS/eip-2200)
        _status = _NOT_ENTERED;
    }
```

`SSTORE` from 0 to 1 (or any non-zero value), the cost is 20000;
`SSTORE` from 1 to 2 (or any other non-zero value), the cost is 5000.

By storing the original value once again, a refund is triggered (https://eips.ethereum.org/EIPS/eip-2200).

Since refunds are capped to a percentage of the total transaction's gas, it is best to keep them low, to increase the likelihood of the full refund coming into effect.

Therefore, switching between 1, 2 instead of 0, 1 will be more gas efficient.

See: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/86bd4d73896afcb35a205456e361436701823c7a/contracts/security/ReentrancyGuard.sol#L29-L33

