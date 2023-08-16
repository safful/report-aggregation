## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Adapt count in setAuctionAverageLookback?](https://github.com/code-423n4/2021-11-malt-findings/issues/194) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function setAuctionAverageLookback of AuctionBurnReserveSkew.sol change auctionAverageLookback 
However there is also the variable "count" that is used in amongst others, addAbovePegObservation().
The modulo of count with auctionAverageLookback is calculated via _getIndexOfObservation().
When you change auctionAverageLookback then the modulo will result in a different value, so you end up in a different location of the circular buffer.

You should probably adapt count as well in the function setAuctionAverageLookback()
(see also function setSampleMemory of MovingAverage.sol where a similar pattern is used)

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/AuctionBurnReserveSkew.sol#L186-L200

```JS
function setAuctionAverageLookback(uint256 _lookback) external onlyRole(ADMIN_ROLE, "Must have admin role") {
..
    if (_lookback > auctionAverageLookback) {
      for (uint i = auctionAverageLookback; i < _lookback; i++) {
        pegObservations.push(0);
      }
    }

    auctionAverageLookback = _lookback;
 ...
```

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/AuctionBurnReserveSkew.sol#L143-L153
```JS
 function addAbovePegObservation(uint256 amount)  public onlyRole(STABILIZER_NODE_ROLE, "Must be a stabilizer node to call this method") {
    uint256 index = _getIndexOfObservation(count);
    ...
    pegObservations[index] = 1;
    count = count + 1;
...
```

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/AuctionBurnReserveSkew.sol#L134-L136
```JS
 function _getIndexOfObservation(uint _index) internal view returns (uint index) {
    return _index % auctionAverageLookback;
  }
```

## Tools Used

## Recommended Mitigation Steps
Doublecheck the theory above and if you agree:
Add the following statement in the function  setAuctionAverageLookback(), before auctionAverageLookback is updated.

```JS
 count = count  % auctionAverageLookback ; // the old version of auctionAverageLookback 
```


