## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Dao.sol: Unbounded Iterations in claimAllForMember()](https://github.com/code-423n4/2021-07-spartan-findings/issues/37) 

# Handle

hickuphh3


# Vulnerability details

### Impact

The `claimAllForMember()` function iterates through the full list of `listedAssets`. Should `listedAssets` become too large, as more assets are listed, calling this function will run out of gas and fail.

### Recommended Mitigation Steps

A good compromise would be to take in an array of asset indexes, so that users can claim for multiple assets in multiple parts.

```jsx
function claimAllForMember(address member, uint256[] calldata assetIndexes)  external returns (bool){
        address [] memory listedAssets = listedBondAssets; // Get array of bond assets
        for(uint i = 0; i < assetIndexes.length; i++){
            uint claimA = calcClaimBondedLP(member, listedAssets[assetIndexes[i]]); // Check user's unlocked Bonded LPs for each asset
            if(claimA > 0){
               _BONDVAULT.claimForMember(listedAssets[assetIndexes[i]], member); // Claim LPs if any unlocked
            }
        }
        return true;
    }
```

