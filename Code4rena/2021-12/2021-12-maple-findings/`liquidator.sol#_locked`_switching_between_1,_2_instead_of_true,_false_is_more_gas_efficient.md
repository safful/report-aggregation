## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`Liquidator.sol#_locked` Switching between 1, 2 instead of true, false is more gas efficient](https://github.com/code-423n4/2021-12-maple-findings/issues/64) 

# Handle

WatchPug


# Vulnerability details

https://github.com/maple-labs/liquidations/blob/bb09e17b1fac1126ce7734e58c3133be06162590/contracts/Liquidator.sol#L45-L62

```solidity
function liquidatePortion(uint256 swapAmount_, uint256 maxReturnAmount_, bytes calldata data_) external override {
    require(!_locked, "LIQ:LP:LOCKED");

    _locked = true;
    ...
    _locked = false;
}
```

`SSTORE` from false (0) to true (1) (or any non-zero value), the cost is 20000;
`SSTORE` from 1 to 2 (or any other non-zero value), the cost is 5000.

By storing the original value once again, a refund is triggered (https://eips.ethereum.org/EIPS/eip-2200).

Since refunds are capped to a percentage of the total transaction's gas, it is best to keep them low, to increase the likelihood of the full refund coming into effect.

Therefore, switching between 1, 2 instead of 0, 1 will be more gas efficient.

See: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/86bd4d73896afcb35a205456e361436701823c7a/contracts/security/ReentrancyGuard.sol#L29-L33

### Recommendation

Change to:

```solidity
function liquidatePortion(uint256 swapAmount_, uint256 maxReturnAmount_, bytes calldata data_) external override {
    require(_locked = 1, "LIQ:LP:LOCKED");

    _locked = 2;
    ...
    _locked = 1;
}
```

