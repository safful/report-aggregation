## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [For uint `> 0` can be replaced with ` != 0` for gas optimization](https://github.com/code-423n4/2022-01-timeswap-findings/issues/172) 

# Handle

0x0x0x


# Vulnerability details

## Impact

`!= 0` is a cheaper operation compared to `> 0`, when dealing with `uint`.

## Occurrences

```

./Timeswap-V1-Convenience/contracts/base/ERC721.sol:147:            } else if (_return.length > 0) {
./Timeswap-V1-Convenience/contracts/libraries/Burn.sol:40:        if (tokensOut.asset > 0) {
./Timeswap-V1-Convenience/contracts/libraries/Burn.sol:65:        if (tokensOut.collateral > 0) {
./Timeswap-V1-Convenience/contracts/libraries/DateTime.sol:128:        if (year >= 1970 && month > 0 && month <= 12) {
./Timeswap-V1-Convenience/contracts/libraries/DateTime.sol:130:            if (day > 0 && day <= daysInMonth) {
./Timeswap-V1-Convenience/contracts/libraries/Mint.sol:296:        require(pair.totalLiquidity(params.maturity) > 0, 'E507');
./Timeswap-V1-Convenience/contracts/libraries/Mint.sol:455:        require(pair.totalLiquidity(params.maturity) > 0, 'E507');
./Timeswap-V1-Convenience/contracts/libraries/Mint.sol:614:        require(pair.totalLiquidity(params.maturity) > 0, 'E507');
./Timeswap-V1-Convenience/contracts/libraries/Pay.sol:86:        if (collateralOut > 0) {
./Timeswap-V1-Convenience/contracts/libraries/PayMath.sol:27:                if (due.debt > 0) {
./Timeswap-V1-Convenience/contracts/libraries/SquareRoot.sol:21:        if (z % y > 0) z++;
./Timeswap-V1-Convenience/contracts/libraries/Withdraw.sol:40:        if (tokensOut.asset > 0) {
./Timeswap-V1-Convenience/contracts/libraries/Withdraw.sol:58:        if (tokensOut.collateral > 0) {
./Timeswap-V1-Convenience/contracts/libraries/Withdraw.sol:75:        if (params.claimsIn.bond > 0)
./Timeswap-V1-Convenience/contracts/libraries/Withdraw.sol:77:        if (params.claimsIn.insurance > 0)
./Timeswap-V1-Core/contracts/TimeswapPair.sol:153:        require(xIncrease > 0 && yIncrease > 0 && zIncrease > 0, 'E205');
./Timeswap-V1-Core/contracts/TimeswapPair.sol:170:        require(liquidityOut > 0, 'E212');
./Timeswap-V1-Core/contracts/TimeswapPair.sol:203:        require(liquidityIn > 0, 'E205');
./Timeswap-V1-Core/contracts/TimeswapPair.sol:217:        if (tokensOut.asset > 0) asset.safeTransfer(assetTo, tokensOut.asset);
./Timeswap-V1-Core/contracts/TimeswapPair.sol:218:        if (tokensOut.collateral > 0) collateral.safeTransfer(collateralTo, tokensOut.collateral);
./Timeswap-V1-Core/contracts/TimeswapPair.sol:236:        require(xIncrease > 0, 'E205');
./Timeswap-V1-Core/contracts/TimeswapPair.sol:239:        require(pool.state.totalLiquidity > 0, 'E206');
./Timeswap-V1-Core/contracts/TimeswapPair.sol:274:        require(claimsIn.bond > 0 || claimsIn.insurance > 0, 'E205');
./Timeswap-V1-Core/contracts/TimeswapPair.sol:292:        if (tokensOut.asset > 0) asset.safeTransfer(assetTo, tokensOut.asset);
./Timeswap-V1-Core/contracts/TimeswapPair.sol:293:        if (tokensOut.collateral > 0) collateral.safeTransfer(collateralTo, tokensOut.collateral);
./Timeswap-V1-Core/contracts/TimeswapPair.sol:311:        require(xDecrease > 0, 'E205');
./Timeswap-V1-Core/contracts/TimeswapPair.sol:314:        require(pool.state.totalLiquidity > 0, 'E206');
./Timeswap-V1-Core/contracts/TimeswapPair.sol:369:        if (assetIn > 0) Callback.pay(asset, assetIn, data);
./Timeswap-V1-Core/contracts/TimeswapPair.sol:374:        if (collateralOut > 0) collateral.safeTransfer(to, collateralOut);
./Timeswap-V1-Core/contracts/libraries/FullMath.sol:34:                require(denominator > 0);
./Timeswap-V1-Core/contracts/libraries/FullMath.sol:124:        if (mulmod(a, b, denominator) > 0) result++;
./Timeswap-V1-Core/contracts/libraries/Math.sol:7:        if (x % y > 0) z++;

```

