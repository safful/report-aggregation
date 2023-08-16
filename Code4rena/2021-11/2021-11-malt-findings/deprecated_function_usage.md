## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Deprecated Function Usage](https://github.com/code-423n4/2021-11-malt-findings/issues/18) 

# Handle

defsec


# Vulnerability details

## Impact

After pragma version, 0.7.0, the contract should use block.timestamp.

## Proof of Concept

1. Navigate to the following contract.

"https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/DexHandlers/UniswapHandler.sol#L153"

"https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/DexHandlers/UniswapHandler.sol#L178"

2. Now is used instead of block.timestamp.

## Tools Used

None

## Recommended Mitigation Steps

It is recommended to use block.timestamp instead of now.

