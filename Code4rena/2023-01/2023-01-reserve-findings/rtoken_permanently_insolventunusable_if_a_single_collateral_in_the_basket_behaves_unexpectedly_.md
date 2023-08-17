## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-16

# [RToken permanently insolvent/unusable if a single collateral in the basket behaves unexpectedly ](https://github.com/code-423n4/2023-01-reserve-findings/issues/254) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/plugins/assets/CTokenFiatCollateral.sol#L45
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/plugins/assets/CTokenFiatCollateral.sol#L37
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/AssetRegistry.sol#L50
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/BasketHandler.sol#L300
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/AssetRegistry.sol#L87


# Vulnerability details

## Impact

Asset plugins assume underlying collateral tokens will always behave as they are expected at the time of the plugin creation. This assumption can be incorrect because of multiple reasons such as upgrades/rug pulls/hacks.

In case a single collateral token in a basket of assets causes functions in the asset to fail the whole RToken functionality will be broken.
This includes (and not limited to):
1. Users cannot redeem RTokens for any collateral
2. Users cannot issue RTokens
3. Bad collateral token cannot be unregistered
4. Stakers will not be able to unstake
5. Recollateralization will not be possible
6. Basket cannot be updated

The impacts become permanent as the unregistering of bad collateral assets is also dependent on collateral token behavior.

Emphasis of funds lost:
A basket holds 2 collateral assets [cAssetA, cAssetB] where cAssetA holds 1% of the RToken collateral and cAssetB holds 99%.
cAssetA gets hacked and self-destructed. This means it will revert on any interaction with it. 
Even though 99% of funds still exists in cAssetB. They will be permanently locked and RToken will be unusable 

## Proof of Concept

Lets assume a `CTokenFiatCollateral` of `cUSDP` is registered as an asset in `AssetRegistry`.
One day, `cUSDP` deployer gets hacked and the contract self-destructs, therefore any call to the `cUSDP` contract will fail. 
`cUSDP` is a proxy contract:
https://etherscan.io/address/0x041171993284df560249B57358F931D9eB7b925D#readProxyContract

Note: There could be other reasons that calls to `cUSDP` will revert such as: 
1. Upgrade to implementation to change/deprecate functions
2. Freezing of contract for a long duration of time (due to patching)
3. blacklisting/whitelisitng callers. 

### Bad collateral assets cannot be unregistered

Lets describe the flow of unregistering an asset from the `AssetRegistry`:
`governance` needs to call `unregister` in order to unregister and asset:
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/AssetRegistry.sol#L87
```
    function unregister(IAsset asset) external governance {
        require(_erc20s.contains(address(asset.erc20())), "no asset to unregister");
        require(assets[asset.erc20()] == asset, "asset not found");
        uint192 quantity = basketHandler.quantity(asset.erc20());

        _erc20s.remove(address(asset.erc20()));
        assets[asset.erc20()] = IAsset(address(0));
        emit AssetUnregistered(asset.erc20(), asset);

        if (quantity > 0) basketHandler.disableBasket();
    }
```

As can seen above, `basketHandler.quantity(asset.erc20());` is called as part of the unregister flow.
`quantity` function in `basketHandler`:
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/BasketHandler.sol#L300
```
    function quantity(IERC20 erc20) public view returns (uint192) {
        try assetRegistry.toColl(erc20) returns (ICollateral coll) {
            if (coll.status() == CollateralStatus.DISABLED) return FIX_ZERO;

            uint192 refPerTok = coll.refPerTok(); // {ref/tok}
            if (refPerTok > 0) {
                // {tok/BU} = {ref/BU} / {ref/tok}
                return basket.refAmts[erc20].div(refPerTok, CEIL);
            } else {
                return FIX_MAX;
            }
        } catch {
            return FIX_ZERO;
        }
    }
```

The asset is still registered so the `try` call will succeed and `coll.refPerTok(); ` will be called. 
`refPerTok` function in `CTokenFiatCollateral` (which is used as an asset of `cUSDP`):
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/plugins/assets/CTokenFiatCollateral.sol#L45
```
    function refPerTok() public view override returns (uint192) {
        uint256 rate = ICToken(address(erc20)).exchangeRateStored();
        int8 shiftLeft = 8 - int8(referenceERC20Decimals) - 18;
        return shiftl_toFix(rate, shiftLeft);
    }
```

if `ICToken(address(erc20)).exchangeRateStored();` will revert because of the previously defined reasons (hack, upgrade, etc..), the whole `unregister` call will be a reverted.

### Explaination of impact

As long as the asset is registered and cannot be removed (explained above), many function calls will revert and cause the impacts in the `impact` section.

The main reason is the `refresh` function of `CTokenFiatCollateral` (used for `cUSDP`) depends on a call to `cUSDP` `exchangeRateCurrent` function. 
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/plugins/assets/CTokenFiatCollateral.sol#L37
```
    function refresh() public virtual override {
        // == Refresh ==
        // Update the Compound Protocol
        ICToken(address(erc20)).exchangeRateCurrent();

        // Intentional and correct for the super call to be last!
        super.refresh(); // already handles all necessary default checks
    }
```  

`AssetRegistry`s `refresh` function calls refresh to all registered assets:
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/AssetRegistry.sol#L50
```
    function refresh() public {
        // It's a waste of gas to require notPausedOrFrozen because assets can be updated directly
        uint256 length = _erc20s.length();
        for (uint256 i = 0; i < length; ++i) {
            assets[IERC20(_erc20s.at(i))].refresh();
        }
    }
```

In our case, `CTokenFiatCollateral.refresh()` will revert therefore the call to `AssetRegistry.refresh()` will revert.

`AssetRegistry.refresh()` is called in critical functions that will revert:
1. `_manageTokens` - used manage backing policy, handout excess assets and perform recollateralization (https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/BackingManager.sol#L107)
2. `refreshBucket` - used to switch the basket configuration (https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/BasketHandler.sol#L184)
3. `issue` - used to issue RTokens to depositors (https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/RToken.sol#L194)
4. `vest` - used to vest issuance of an account (https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/RToken.sol#L380)
5. `redeem` - used to redeem collateral assets for RTokens (https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/RToken.sol#L443)
6. `poke` - in main, used as a refresher (https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/Main.sol#L45)
7. `withdraw` in RSR, stakers will not be able to unstake (https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L302)

## Tools Used

Foundry, VS Code

## Recommended Mitigation Steps

For plugins to function as intended there has to be a dependency on protocol specific function.
In a case that the collateral token is corrupted, the governance should be able to replace to corrupted token. The unregistering flow should never be depended on the token functionality.  