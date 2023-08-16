## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Initialization Function Is Missing If Token is Equals To WAVAX On the LaunchEvent](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/64) 

# Handle

defsec


# Vulnerability details

## Impact

During the code review, It has been observed that the token can be same as WAVAX. The initialize function should not allow if token is equals to wavax. That would affect all asset management.


## Proof of Concept

1. Navigate to the following contract.

```
https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L219

https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L411

```

## Tools Used

Code Review

## Recommended Mitigation Steps

On the Launchevent, token should not be equal to wavax. 

