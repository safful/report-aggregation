## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [PrizePool.beforeTokenTransfer() incorrectly uses msg.sender in seven places instead of _msgSender()](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/23) 

# Handle

jvaqa


# Vulnerability details

## Impact
PrizePool.beforeTokenTransfer() incorrectly uses msg.sender in seven places instead of _msgSender(). [1]

Nearly all of PrizePool.sol opts to use _msgSender() to provide for more optionality. 

It appears that PrizePool.beforeTokenTransfer() may have been copy/pasted into PrizePool.sol without adjusting msg.sender to use _msgSender().

## Recommended Mitigation Steps

Replace the seven instances of msg.sender in PrizePool.beforeTokenTransfer() with _msgSender()

[1] https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L418

