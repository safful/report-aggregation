## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [Fund stuck in `Liquidator` if `stopLiquidation` is called ](https://github.com/code-423n4/2021-12-maple-findings/issues/67) 

# Handle

gzeon


# Vulnerability details

## Impact
`stopLiquidation` does not pull fund from `Liquidator` while setting `_liquidator` to address(0). Since the `DebtLocker` own `Liquidator` and there are no way to set `_liquidator` to existing address, fund still in `Liquidator` would be stuck unless `DebtLocker` is upgraded to support such behavior.

Also, when `stopLiquidation` is called, remaining fund in Liquidator can still be liquidated by keepers.

## Proof of Concept
https://github.com/maple-labs/debt-locker/blob/81f55907db7b23d27e839b9f9f73282184ed4744/contracts/DebtLocker.sol#L112
```
    function stopLiquidation() external override {
        require(msg.sender == _getPoolDelegate(), "DL:SL:NOT_PD");

        _liquidator = address(0);

        emit LiquidationStopped();
    }
```

## Recommended Mitigation Steps
Pull remaining fund in `stopLiquidation`  

