## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Recommend to use OZ SafeERC20 library](https://github.com/code-423n4/2021-10-covalent-findings/issues/1) 

# Handle

xYrYuYx


# Vulnerability details

## Impact
This is too complicated steps to transfer ERC20 token which could use more gas.
You don't need to check balance before transfer.
If there is no enough balance, it SafeERC20 will revert.
Also you don't need to check balance after transfer, because CQT does not have transaction fee.

## Proof of Concept
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol

## Tools Used

## Recommended Mitigation Steps
Since there is no transaction fee in CQT token, you can use OZ SafeERC20 library to send or receive.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol#L20
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol#L28


