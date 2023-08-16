## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Open TODOs in Codebase](https://github.com/code-423n4/2021-11-overlay-findings/issues/116) 

# Handle

pauliax


# Vulnerability details

## Impact
There are TODOs left in the code. While this does not cause any direct issue, it indicates a bad smell and uncertainty. In previous reports, such submissions were assigned a score of 'low' so I think it's a fair game to submit this as an issue here also. 
Reference:
https://github.com/code-423n4/2021-09-swivel-findings/issues/67
https://github.com/code-423n4/2021-10-tempus-findings/issues/39

Also, there are some misleading comments, e.g.:
```solidity
    /// @notice Internal update function to price, cap, and pay funding.
    function update () public virtual returns (
```
the comment says that function is internal but it is actually declared as public.

## Recommended Mitigation Steps
Consider implementing or removing TODOs and updating misleading comments.

