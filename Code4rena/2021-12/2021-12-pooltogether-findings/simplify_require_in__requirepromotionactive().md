## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [simplify require in _requirePromotionActive()](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/19) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function _requirePromotionActive() contains the following check in a require statement:
```JS
 _promotionEndTimestamp > 0 && _promotionEndTimestamp >= block.timestamp,
```
When _promotionEndTimestamp is larger than block.timestamp it will also be larger than 0.
Thus the statement can be simplified to save some gas.

## Proof of Concept
https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L250-L258

```JS
function _requirePromotionActive(Promotion memory _promotion) internal view {
        ...
        require(  _promotionEndTimestamp > 0 && _promotionEndTimestamp >= block.timestamp, "TwabRewards/promotion-not-active" );
    }
```


## Tools Used

## Recommended Mitigation Steps
Change the require statement to:
        require( _promotionEndTimestamp >= block.timestamp, "TwabRewards/promotion-not-active" ); // will certainly be > 0

