## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Use of Large Number Literals](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/64) 

# Handle

SolidityScan


# Vulnerability details

### Description

Integer literals are formed from a sequence of digits in the range 0-9. They are interpreted as decimals. The use of very large numbers with too many digits was detected in the code that could have been optimized using a different notation also supported by Solidity.

## Impact
Literals with many digits are difficult to read and review. This may also introduce errors in the future if one of the zeroes is omitted while doing code modifications. 

## Proof of Concept
1.  uint constant public _100pct = 1000000000000000000; 
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Dependencies/LiquityBase.sol#L19

2. uint constant public _110pct = 1100000000000000000; // 1.1e18 == 110%

https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Dependencies/LiquityBase.sol#L21

3. uint constant public MCR = 1100000000000000000; // 110%

https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Dependencies/LiquityBase.sol#L24

4. uint constant public CCR = 1500000000000000000; // 150%
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Dependencies/LiquityBase.sol#L27

## Tools Used

## Recommended Mitigation Steps
Scientific notation in the form of 2e10 is also supported, where the mantissa can be fractional but the exponent has to be an integer. The literal MeE is equivalent to M * 10**E. Examples include 2e10, 2e10, 2e-10, 2.5e1.
As suggested in official docs
https://docs.soliditylang.org/en/latest/types.html#rational-and-integer-literals

