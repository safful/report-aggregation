## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Deprecated Function Usage](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/69) 

# Handle

defsec


# Vulnerability details

## Impact
After pragma version, 0.7.0, the contract should use block.timestamp.

## Proof of Concept

1. Navigate to the following contract.

"https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtcEth.sol#L74"

2. Now is used instead of block.timestamp.

## Tools Used

None

## Recommended Mitigation Steps

It is recommended to use block.timestamp instead of now. 

