## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [cancelPromotion is too rigorous](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/23) 

# Handle

gpersoon


# Vulnerability details

## Impact
When you cancel a promotion with cancelPromotion() then the promotion is complete deleted.
This means no-one can claim any rewards anymore, because  _promotions[_promotionId] no longer exists.

It also means all the unclaimed tokens (of the previous epochs) will stay locked in the contract.

## Proof of Concept
https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L119-L138
```JS
function cancelPromotion(uint256 _promotionId, address _to) ... {
       ...
        uint256 _remainingRewards = _getRemainingRewards(_promotion);
        delete _promotions[_promotionId];
       
```

## Tools Used

## Recommended Mitigation Steps
In the function cancelPromotion() lower the numberOfEpochs or set a state variable, to allow user to claim their rewards.

