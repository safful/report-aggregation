## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [EthAssetManager and ThreePoolAssetManager don't control Meta tokens decimals](https://github.com/code-423n4/2022-05-alchemix-findings/issues/63) 

# Lines of code

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/ThreePoolAssetManager.sol#L896-L905
https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/EthAssetManager.sol#L566-L573


# Vulnerability details

Both contracts treat meta assets as if they have fixed decimals of 18. Minting logic breaks when it's not the case. However, meta tokens decimals aren't controlled.

If actual meta assets have any other decimals, minting slippage control logic of both contracts will break up as `total` is calculated as a plain sum of token amounts.

In the higher token decimals case `minTotalAmount` will be magnitudes higher than actual amount Curve can provide and minting becomes unavailable.

In the lower token decimals case `minTotalAmount` will lack value and slippage control will be rendered void, which opens up a possibility of a fund loss from the excess slippage.

Setting severity to medium as the contract can be used with various meta tokens (`_metaPoolAssetCache`  can be filled with any assets) and, whenever decimals differ from 18 `add_liquidity` uses, its logic be broken: the inability to mint violates the contract purpose, the lack of slippage control can lead to fund losses.

I.e. this is system breaking impact conditional on a low probability assumption of different meta token decimals.

## Proof of Concept

Meta tokens decimals are de facto hard coded into the contract as plain amounts are used (L. 905):

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/ThreePoolAssetManager.sol#L896-L905

```solidity
    function _mintMetaPoolTokens(
        uint256[NUM_META_COINS] calldata amounts
    ) internal returns (uint256 minted) {
        IERC20[NUM_META_COINS] memory tokens = _metaPoolAssetCache;

        uint256 total = 0;
        for (uint256 i = 0; i < NUM_META_COINS; i++) {
            if (amounts[i] == 0) continue;

            total += amounts[i];
```

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/ThreePoolAssetManager.sol#L915-L919

```solidity
        uint256 expectedOutput    = total * CURVE_PRECISION / metaPool.get_virtual_price();
        uint256 minimumMintAmount = expectedOutput * metaPoolSlippage / SLIPPAGE_PRECISION;

        // Add the liquidity to the pool.
        minted = metaPool.add_liquidity(amounts, minimumMintAmount);
```

The same plain sum approach is used in EthAssetManager._mintMetaPoolTokens:

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/EthAssetManager.sol#L566-L573

```solidity
        uint256 total = 0;
        for (uint256 i = 0; i < NUM_META_COINS; i++) {
            // Skip over approving WETH since we are directly swapping ETH.
            if (i == uint256(MetaPoolAsset.ETH)) continue;

            if (amounts[i] == 0) continue;

            total += amounts[i];
```

When this decimals assumption doesn't hold, the slippage logic will not hold too: either the mint be blocked or slippage control disabled.

Notice, that ThreePoolAssetManager.calculateRebalance do query alUSD decimals (which is inconsistent with the above as it’s either fix and control on inception or do not fix and accommodate the logic):

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/ThreePoolAssetManager.sol#L338-L338

```solidity
decimals     = SafeERC20.expectDecimals(address(alUSD));
```

## Recommended Mitigation Steps

If meta assets are always supposed to have fixed decimals of 18, consider controlling it at the construction time.

I.e. the decimals can be controlled in constructors:

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/EthAssetManager.sol#L214-L219

```solidity
        for (uint256 i = 0; i < NUM_META_COINS; i++) {
            _metaPoolAssetCache[i] = params.metaPool.coins(i);
            if (_metaPoolAssetCache[i] == IERC20(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE)) {
                _metaPoolAssetCache[i] = weth;
+           } else {
+           	// check the decimals
				}
        }
```

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/ThreePoolAssetManager.sol#L254-L256

```solidity
        for (uint256 i = 0; i < NUM_META_COINS; i++) {
            _metaPoolAssetCache[i] = params.metaPool.coins(i);
+           // check the decimals            
        }
```

In this case further decimals reading as it's done in calculateRebalance() is redundant.

Otherwise (which is less recommended as fixed decimals assumption is viable and simplify the logic) the meta token decimals can be added to calculations similarly to stables:

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/ThreePoolAssetManager.sol#L779-L779

```solidity
normalizedTotal += amounts[i] * 10**missingDecimals;
```

