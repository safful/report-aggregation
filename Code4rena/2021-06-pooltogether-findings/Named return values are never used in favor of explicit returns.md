## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Named return values are never used in favor of explicit returns](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/53) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Named return values in multiple functions are never used in favor of explicit returns. 

Impact: This affects readability/auditability at the least and could potentially result in unexpected values being returned along paths with no explicit returns.

## Proof of Concept

Unused in favor of explicit return: 

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L717-L726

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L741-L744

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L770

Used without explicit return:

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L923-L930

Used with explicit return:

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L944-L947


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove unused named returns where unnecessary. Be consistent in using named vs explicit returns.

