## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Superfluous verifySignature function](https://github.com/code-423n4/2021-06-tracer-findings/issues/121) 

# Handle

0xsanson


# Vulnerability details

## Impact
In the trader contract isValidSignature(...) and verifySignature(...) serve the same purpose. Suggested keep only one for code clarity.

## Proof of Concept
https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Trader.sol#L206
https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Trader.sol#L231

## Tools Used
Manual analysys

## Recommended Mitigation Steps
Suggested keep only one function for code clarity.

