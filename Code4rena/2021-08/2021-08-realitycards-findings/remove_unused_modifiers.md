## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- Resolved

# [remove unused modifiers](https://github.com/code-423n4/2021-08-realitycards-findings/issues/10) 

# Handle

gpersoon


# Vulnerability details

## Impact
Several modifiers are defined, but not used:
- onlyTokenOwner in RCMarket.sol
- onlyFactory in RCOrderbook.sol and RCLeaderboard.sol

This clutters the code base

## Proof of Concept
// https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCMarket.sol#L328
    modifier onlyTokenOwner(uint256 _token) {
    ...

//https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCOrderbook.sol#L104
  modifier onlyFactory() {
    ...

//https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCLeaderboard.sol#L61
  modifier onlyFactory() {
  ...

## Tools Used

## Recommended Mitigation Steps
Remove the unused modifiers


