## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [Return Value is Not Validated](https://github.com/code-423n4/2021-08-realitycards-findings/issues/24) 

# Handle

leastwood


# Vulnerability details

## Impact
The `circuitBreaker()` function in `RCMarket.sol` is utilised in the event an oracle never provides a response to a RealityCards question. The function makes an external call to the `RCOrderbook.sol` contract through the `closeMarket()` function. If for some reason the orderbook was unable to be closed, this would never be checked in the `circuitBreaker()` function.

## Proof of Concept

https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCMarket.sol#L1215-L1223

## Tools Used

Manual code review

## Recommended Mitigation Steps

Ensure this is intended behaviour, or otherwise validate the response of `orderbook.closeMarket()`. Another option would be to emit the result of the external call in the `LogStateChange` event, alongside the state change.

