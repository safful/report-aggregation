## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- old-submission-method

# [Solmate safetransfer and safetransferfrom doesnot check the codesize of the token address, which may lead to fund loss](https://github.com/code-423n4/2022-08-olympus-findings/issues/117) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/TRSRY.sol#L110
https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/TRSRY.sol#L99


# Vulnerability details

## Impact
In `getloan()` and `replayloan()`, the `safetransfer` and `safetransferfrom` doesn't check the existence of code at the token address. This is a known issue while using solmate's libraries. 
Hence this may lead to miscalculation of funds and may lead to loss of funds , because if `safetransfer()` and `safetransferfrom()` are called on a token address that doesn't have contract in it, it will always return success, bypassing the return value check. Due to this protocol will think that funds has been transferred and successful , and records will be accordingly calculated, but in reality funds were never transferred. 
So this will lead to miscalculation and possibly loss of funds

## Proof of Concept
https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/TRSRY.sol#L110
https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/TRSRY.sol#L99

## Tools Used
Manual code review

## Recommended Mitigation Steps
Use openzeppelin's safeERC20 or implement a code existence check

