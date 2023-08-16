## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [TwarbRewards: don't use the onlyPromotionCreator modifier to save gas](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/77) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
cancelPromotion() and its modifier both call _getPromotion() to get the Promotion struct. We can save one such call by removing the modifier and do the check of the modifier at the beginning of the cancelPromotion() block to save storage reads.


## Proof of Concept
https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L119

## Tools Used

## Recommended Mitigation Steps
- remove the modifier onlyPromotionCreator
- do the require statement at the beginning of cancelPromotion()


    function cancelPromotion(uint256 _promotionId, address _to)
        external
        override
        returns (bool)
    {
        Promotion memory _promotion = _getPromotion(_promotionId);
       
       // do here the modifiers check
         require(
            msg.sender == _promotion .creator,
            "TwabRewards/only-promotion-creator"
        );

        _requirePromotionActive(_promotion);
        require(_to != address(0), "TwabRewards/recipient-not-zero-address");

        uint256 _remainingRewards = _getRemainingRewards(_promotion);

        delete _promotions[_promotionId];
        _promotion.token.safeTransfer(_to, _remainingRewards);

        emit PromotionCancelled(_promotionId, _remainingRewards);

        return true;
    }

