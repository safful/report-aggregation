## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [_getPromotion() doesn't revert on invalid _promotionId](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/2) 

# Handle

johnnycash


# Vulnerability details

## Impact

`_getPromotion()` doesn't revert if the specified `_promotionId` doesn't exist. It can lead to unexpected behaviors in callers of this function.

For instance, [claimRewards](https://github.com/pooltogether/v4-periphery/blob/ceadb25844f95f19f33cb856222e461ed8edf005/contracts/TwabRewards.sol#L162) will continue its execution and call `_calculateRewardAmount()` and eventually `_promotion.token.safeTransfer()` (probably with `_rewardsAmount` equal to 0).


## Analysis

In contrary to the following comment:

```
@dev Will revert if the promotion does not exist.
```

[_getPromotion()](https://github.com/pooltogether/v4-periphery/blob/ceadb25844f95f19f33cb856222e461ed8edf005/contracts/TwabRewards.sol#L260-L268) doesn't revert if the specified `_promotionId` doesn't exist, but return a `Promotion` structure with all fields set to 0.


## Tools Used

Text editor.


## Recommended Mitigation Steps

Fix suggestion:

```
    function _getPromotion(uint256 _promotionId) internal view returns (Promotion memory _promotion) {
        _promotion = _promotions[_promotionId];
        require(_promotion.creator != address(0), "TwabRewards/invalid-promotion");
        return _promotion;
    }
```

