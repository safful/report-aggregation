## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- Resolved

# [gas saving in `_processRentCollection`](https://github.com/code-423n4/2021-08-realitycards-findings/issues/68) 

# Handle

0xsanson


# Vulnerability details

## Impact
In `rcMarket._processRentCollection` it's possible to save a SLOAD by rewriting the lines:
```
uint256 _rentOwed = (card[_card].cardPrice *
    (_timeOfCollection - card[_card].timeLastCollected)) / 1 days;
uint256 _timeHeldToIncrement = (_timeOfCollection -               
    card[_card].timeLastCollected);
```
into:
```
uint256 _timeHeldToIncrement = (_timeOfCollection -               
    card[_card].timeLastCollected);
uint256 _rentOwed = (card[_card].cardPrice * _timeHeldToIncrement) / 1 days;
```

## Proof of Concept
https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCMarket.sol#L1060-L1063

## Tools Used
editor

## Recommended Mitigation Steps
Consider changing the code as illustrated.

