## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [OverlayV1OVLCollateral.liquidate storage pos.market variable is read up to three times, can be saved to memory](https://github.com/code-423n4/2021-11-overlay-findings/issues/138) 

# Handle

hyh


# Vulnerability details

## Impact

Gas is overspent on storage access.

## Proof of Concept

```pos.market``` variable is being read up to three times from storage:
https://github.com/code-423n4/2021-11-overlay/blob/main/contracts/collateral/OverlayV1OVLCollateral.sol#L371

## Recommended Mitigation Steps

Save the needed storage variable to memory and use it.

Now:
```
Position.Info storage pos = positions[_positionId];
...

(   uint _oi,
		uint _oiShares,
		uint _priceFrame ) = IOverlayV1Market(pos.market)
				.exitData(
						_isLong,
						pos.pricePoint
				);

MarketInfo memory _marketInfo = marketInfo[pos.market];
...
IOverlayV1Market(pos.market).exitOI(...
```

To be:
```
Position.Info storage pos = positions[_positionId];
address memory pos_market = pos.market;
...

(   uint _oi,
		uint _oiShares,
		uint _priceFrame ) = IOverlayV1Market(pos_market)
				.exitData(
						_isLong,
						pos.pricePoint
				);

MarketInfo memory _marketInfo = marketInfo[pos_market];
...
IOverlayV1Market(pos_market).exitOI(...
```

