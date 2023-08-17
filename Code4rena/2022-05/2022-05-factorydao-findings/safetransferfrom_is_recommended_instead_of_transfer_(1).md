## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [safeTransferFrom is recommended instead of transfer (1)](https://github.com/code-423n4/2022-05-factorydao-findings/issues/22) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/e22a562c01c533b8765229387894cc0cb9bed116/contracts/PermissionlessBasicPoolFactory.sol#L144


# Vulnerability details

## Impact 
ERC20 standard allows transferF function of some contracts to return bool or return nothing.
Some tokens such as USDT return nothing.
This could lead to funds stuck in the contract without possibility to retrieve them.
Using safeTransferFrom of SafeERC20.sol is recommended instead.

## Proof of Concept 
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4a9cc8b4918ef3736229a5cc5a310bdc17bf759f/contracts/token/ERC20/utils/SafeERC20.sol

## Tools Used 

## Recommended Mitigation Steps

