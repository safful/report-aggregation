## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [event PromotionCancelled should also emit the _to address](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/127) 

# Handle

hubble


# Vulnerability details

## Impact
Since there is an option for the promoter to provide an alternate address while issuing cancelPromotion apart from the creator(promoter address)
It is good to track the _to address where the remainingRewards are sent on cancelPromotion

## Proof of Concept
contract : TwabRewards
line 50 :    event PromotionCancelled(uint256 indexed promotionId, uint256 amount);

function : cancelPromotion(uint256 _promotionId, address _to)
line 135 :  emit PromotionCancelled(_promotionId, _remainingRewards);

## Tools Used
Manual review

## Recommended Mitigation Steps
Add the 'to address' in the event, as below

line 50 :    event PromotionCancelled(uint256 indexed promotionId, address to, uint256 amount);

function : cancelPromotion(uint256 _promotionId, address _to)
line 135 :  emit PromotionCancelled(_promotionId, _to, _remainingRewards);



