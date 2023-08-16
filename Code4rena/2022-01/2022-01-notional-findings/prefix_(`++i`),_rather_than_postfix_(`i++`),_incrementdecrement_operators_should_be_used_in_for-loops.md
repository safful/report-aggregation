## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Prefix (`++i`), rather than postfix (`i++`), increment/decrement operators should be used in for-loops](https://github.com/code-423n4/2022-01-notional-findings/issues/228) 

# Handle

IllIllI


# Vulnerability details

## Impact
When the value of the post-loop increment/decrement is not stored or used in any calculations, the prefix increment/decrement operators (`++i`/`--i`) cost less gas PER LOOP than the postfix increment/decrement operators (`i++`/`i--`)

## Proof of Concept
There is one example of this issue in the codebase:

```Solidity
for (uint256 i; i < currencies.length; i++) {        
```
https://github.com/code-423n4/2022-01-notional/blob/main/contracts/TreasuryAction.sol#L157


## Tools Used
Code inspection

## Recommended Mitigation Steps
Use `++i` rather than `i++` in all places

