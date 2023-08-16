## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [AuctionBurnReserveSkew:addAbovePegObservation gas optimization](https://github.com/code-423n4/2021-11-malt-findings/issues/153) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Remove L151 count = count + 1;
and change L147 to uint256 index = _getIndexOfObservation(count++); so we save at least a SLOAD.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionBurnReserveSkew.sol#L143

## Tools Used

## Recommended Mitigation Steps
Remove L151 count = count + 1;
and change L147 to uint256 index = _getIndexOfObservation(count++);

