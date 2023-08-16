## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [TimeswapPair.pay doesn't check for non-existent debt owner](https://github.com/code-423n4/2022-01-timeswap-findings/issues/97) 

# Handle

hyh


# Vulnerability details

## Impact

Similarly, the system will fail with low-level message without giving a business reason, which can be an issue for troubleshooting and further programmatic usages by other projects. 

## Proof of Concept

The owner variable is used by TimeswapPair.pay without validation:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L357

Which will yield low-level fail on array access if an owner is zero or not present in the system:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L360


## Recommended Mitigation Steps

Verify owner function argument to be non-zero by expanding existing check:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L352

Then, require in line 358 that, for example, dues.length >= ids.length:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L358


