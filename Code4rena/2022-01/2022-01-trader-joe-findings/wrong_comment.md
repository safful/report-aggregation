## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [Wrong comment](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/149) 

# Handle

wuwe1


# Vulnerability details

## Impact
Causing confuse to user and developer.

## Proof of Concept
https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L55

`105000 * 1e18 / (1e18 + 5e16)` is equal to `100000`



## Recommended Mitigation Steps

change to

`105000 - 105000 * 1e18 / (1e18 + 5e16) = 5000`

