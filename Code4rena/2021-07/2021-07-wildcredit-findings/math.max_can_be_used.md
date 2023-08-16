## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Math.max can be used](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/133) 

# Handle

0xsanson


# Vulnerability details

## Impact
The line `return (rate < MIN_RATE) ? MIN_RATE : rate;` can be written as `return Math.max(rate, MIN_RATE);` for an easier reading, since the Math library is already imported.

## Proof of Concept
https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/InterestRateModel.sol#L37

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Rewrite using the Math.max function

