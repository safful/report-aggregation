## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [SafeTransfer library called from pay() function is not needed ](https://github.com/code-423n4/2022-01-timeswap-findings/issues/11) 

# Handle

jayjonah8


# Vulnerability details

## Impact
In TimeswapPair.sol the pay() function calls safeTransfer() and does so using the SafeTransfer.sol library when it can simply add the open zeppelin SafeERC20.sol import directly inside TimeswapPair.sol itself eliminating the unnecessary code in the protocols own SafeTransfer library. 

## Proof of Concept
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L374

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/libraries/SafeTransfer.sol#L7

## Tools Used
Manual code review 

## Recommended Mitigation Steps
Use the open zeppelin SafeERC20 import directly inside the TimeswapPair.sol file instead of calling your own library.  The extra safeTransfer library can then be deleted. 

