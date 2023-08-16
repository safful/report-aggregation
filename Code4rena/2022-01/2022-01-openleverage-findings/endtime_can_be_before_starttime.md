## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [endTime can be before startTime](https://github.com/code-423n4/2022-01-openleverage-findings/issues/160) 

# Handle

samruna


# Vulnerability details

https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/OLETokenLock.sol#L66

In the above code, there is no check to see if endTime is before startTime. Due to this past beneficiaries can be transferred additional tokens

Action:
check if endTime if always in future.

