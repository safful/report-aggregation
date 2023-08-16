## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Avoid unnecessary arithmetic operations can save gas](https://github.com/code-423n4/2021-12-perennial-findings/issues/20) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/product/types/position/PrePosition.sol#L145-L156

```solidity=145
function computeFee(PrePosition memory self, IProductProvider provider, uint256 toOracleVersion) internal view returns (UFixed18 positionFee) {
    Fixed18 oraclePrice = provider.priceAtVersion(toOracleVersion);
    Position memory positionDelta = self.openPosition.add(self.closePosition);

    (UFixed18 makerNotional, UFixed18 takerNotional) = (
        Fixed18Lib.from(positionDelta.maker).mul(oraclePrice).abs(),
        Fixed18Lib.from(positionDelta.taker).mul(oraclePrice).abs()
    );

    positionFee = positionFee.add(makerNotional.mul(provider.safeMakerFee()));
    positionFee = positionFee.add(takerNotional.mul(provider.safeTakerFee()));
}
```

At L154, `positionFee = positionFee.add(makerNotional.mul(provider.safeMakerFee()));`  can be changed to `positionFee = makerNotional.mul(provider.safeMakerFee());` as `positionFee == 0`.

Futhermore, L154-155 can be combined into:

```solidity
positionFee = makerNotional.mul(provider.safeMakerFee()).add(
    takerNotional.mul(provider.safeTakerFee())
);
```

