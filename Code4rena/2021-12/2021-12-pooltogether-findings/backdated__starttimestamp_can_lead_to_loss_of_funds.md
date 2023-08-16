## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [Backdated _startTimestamp can lead to loss of funds](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/8) 

# Handle

csanuragjain


# Vulnerability details

## Impact
This can lead to loss of funds as there is no recovery function of funds stuck like this

## Proof of Concept
1. User A creates a new promotion using createPromotion function. By mistake he provides 1 year ago value for _startTimestamp with promotion duration as 6 months

2. Since there is no check to see that _startTimestamp > block.timestamp so this promotion gets created

3. User cannot claim this promotion if they were not having promotion tokens in the 1 year old promotion period. This means promotion amount remains with contract

4. Even promotion creator cannot claim back his tokens since promotion end date has already passed so cancelPromotion will fail

5. As there is no recovery token function in contract so even contract cant transfer this token and the tokens will remain in this contract with no one able to claim those


## Recommended Mitigation Steps
Add below check in the createPromotion function

```
function createPromotion(
        address _ticket,
        IERC20 _token,
        uint216 _tokensPerEpoch,
        uint32 _startTimestamp,
        uint32 _epochDuration,
        uint8 _numberOfEpochs
    ) external override returns (uint256) {
require(_startTimestamp>block.timestamp,"should be after current time");
}
```

