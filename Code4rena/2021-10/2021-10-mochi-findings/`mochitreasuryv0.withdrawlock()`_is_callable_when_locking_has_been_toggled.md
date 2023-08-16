## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`MochiTreasuryV0.withdrawLock()` Is Callable When Locking Has Been Toggled](https://github.com/code-423n4/2021-10-mochi-findings/issues/161) 

# Handle

leastwood


# Vulnerability details

## Impact

`withdrawLock()` does not prevent users from calling this function when locking has been toggled. As a result, withdraws may be made unexpectedly.

## Proof of Concept

https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/treasury/MochiTreasuryV0.sol#L40-L42

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider adding `require(lockCrv, "!lock");` to `withdrawLock()` to ensure this function is not called unexpectedly. Alternatively if this is intended behaviour, it should be rather checked that the lock has not been toggled, otherwise users could maliciously relock tokens.

