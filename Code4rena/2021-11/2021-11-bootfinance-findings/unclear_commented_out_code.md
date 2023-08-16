## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Unclear Commented Out Code](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/140) 

# Handle

ye0lde


# Vulnerability details

# Vulnerability details

## Impact

I'm not sure why some of this code is commented out.  
It could point to items that are not done or need redesigning, be a mistake, or just be testing overhead. 

## Proof of Concept
The commented out code is here:

Unclear:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/Vesting.sol#L27
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/Vesting.sol#L76

Obviously Test related:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/customswap/contracts/Swap.sol#L187

Guarded launch:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/customswap/contracts/Swap.sol#L42-L49
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/customswap/contracts/Swap.sol#L201-L204
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/customswap/contracts/Swap.sol#L235-L237

## Tools Used
VS Code

## Recommended Mitigation Steps
Review and remove or resolve/document the commented out lines if needed.

