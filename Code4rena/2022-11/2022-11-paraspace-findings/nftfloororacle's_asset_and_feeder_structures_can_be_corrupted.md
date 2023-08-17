## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- upgraded by judge
- H-08

# [NFTFloorOracle's asset and feeder structures can be corrupted](https://github.com/code-423n4/2022-11-paraspace-findings/issues/482) 

# Lines of code

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L278-L286
https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L307-L316


# Vulnerability details

NFTFloorOracle's _addAsset() and _addFeeder() truncate the `assets` and `feeders` arrays indices to 255, both using `uint8 index` field in the corresponding structures and performing `uint8(assets.length - 1)` truncation on the new element addition.

`2^8 - 1` looks to be too tight as an **all time** element count limit. It can be realistically surpassed in a couple years time, especially given multi-asset and multi-feeder nature of the protocol. This way this isn't a theoretical unsafe truncation, but an accounting malfunction that is practically reachable given long enough system lifespan, without any additional requirements as asset/feeder turnaround is a going concern state of the system.

## Impact

Once truncation start corrupting the indices the asset/feeder structures will become incorrectly referenced and removal of an element will start to remove another one, permanently breaking up the structures.

This will lead to inability to control these structures and then to Oracle malfunction. This can lead to collateral mispricing. Setting the severity to be medium due to prerequisites.

## Proof of Concept

`feederPositionMap` and `assetFeederMap` use `uint8` indices:

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L32-L48

```solidity
struct FeederRegistrar {
    // if asset registered or not
    bool registered;
    // index in asset list
    uint8 index;
    // if asset paused,reject the price
    bool paused;
    // feeder -> PriceInformation
    mapping(address => PriceInformation) feederPrice;
}

struct FeederPosition {
    // if feeder registered or not
    bool registered;
    // index in feeder list
    uint8 index;
}
```

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L79-L88

```solidity
    /// @dev feeder map
    // feeder address -> index in feeder list
    mapping(address => FeederPosition) private feederPositionMap;

    ...

    /// @dev Original raw value to aggregate with
    // the NFT contract address -> FeederRegistrar which contains price from each feeder
    mapping(address => FeederRegistrar) public assetFeederMap;
```

On entry removal both `assets` array length do not decrease:

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L296-L305

```solidity
    function _removeAsset(address _asset)
        internal
        onlyWhenAssetExisted(_asset)
    {
        uint8 assetIndex = assetFeederMap[_asset].index;
        delete assets[assetIndex];
        delete assetPriceMap[_asset];
        delete assetFeederMap[_asset];
        emit AssetRemoved(_asset);
    }
```

On the contrary, feeders array is being decreased:

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L326-L338

```solidity
    function _removeFeeder(address _feeder)
        internal
        onlyWhenFeederExisted(_feeder)
    {
        uint8 feederIndex = feederPositionMap[_feeder].index;
        if (feederIndex >= 0 && feeders[feederIndex] == _feeder) {
            feeders[feederIndex] = feeders[feeders.length - 1];
            feeders.pop();
        }
        delete feederPositionMap[_feeder];
        revokeRole(UPDATER_ROLE, _feeder);
        emit FeederRemoved(_feeder);
    }
```

I.e. `assets` array element is set to zero with `delete`, but not removed from the array.

This means that `assets` will only grow over time, and will eventually surpass `2^8 - 1 = 255`. That's realistic given that assets here are NFTs, whose variety will increase over time.

Once this happen the truncation will start to corrupt the indices:

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L278-L286

```solidity
    function _addAsset(address _asset)
        internal
        onlyWhenAssetNotExisted(_asset)
    {
        assetFeederMap[_asset].registered = true;
        assets.push(_asset);
        assetFeederMap[_asset].index = uint8(assets.length - 1);
        emit AssetAdded(_asset);
    }
```

This can happen with `feeders` too, if the count merely surpass `255` with net additions:

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L307-L316

```solidity
    function _addFeeder(address _feeder)
        internal
        onlyWhenFeederNotExisted(_feeder)
    {
        feeders.push(_feeder);
        feederPositionMap[_feeder].index = uint8(feeders.length - 1);
        feederPositionMap[_feeder].registered = true;
        _setupRole(UPDATER_ROLE, _feeder);
        emit FeederAdded(_feeder);
    }
```

This will lead to _removeAsset() and _removeFeeder() clearing another assets/feeders as the `assetFeederMap[_asset].index` and `feederPositionMap[_feeder].index` become broken being truncated. It will permanently mess the structures.

## Recommended Mitigation Steps

As a simplest measure consider increasing the limit to `2^32 - 1`:

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L278-L286

```solidity
    function _addAsset(address _asset)
        internal
        onlyWhenAssetNotExisted(_asset)
    {
        assetFeederMap[_asset].registered = true;
        assets.push(_asset);
-       assetFeederMap[_asset].index = uint8(assets.length - 1);
+       assetFeederMap[_asset].index = uint32(assets.length - 1);
        emit AssetAdded(_asset);
    }
```

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L307-L316

```solidity
    function _addFeeder(address _feeder)
        internal
        onlyWhenFeederNotExisted(_feeder)
    {
        feeders.push(_feeder);
-       feederPositionMap[_feeder].index = uint8(feeders.length - 1);
+       feederPositionMap[_feeder].index = uint32(feeders.length - 1);
        feederPositionMap[_feeder].registered = true;
        _setupRole(UPDATER_ROLE, _feeder);
        emit FeederAdded(_feeder);
    }
```

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/misc/NFTFloorOracle.sol#L32-L48

```solidity
struct FeederRegistrar {
    // if asset registered or not
    bool registered;
    // index in asset list
-   uint8 index;
+   uint32 index;
    // if asset paused,reject the price
    bool paused;
    // feeder -> PriceInformation
    mapping(address => PriceInformation) feederPrice;
}

struct FeederPosition {
    // if feeder registered or not
    bool registered;
    // index in feeder list
-   uint8 index;
+   uint32 index;
}
```

Also, consider actually removing `assets` array element in _removeAsset() via the usual moving of the last element as it's done in _removeFeeder().