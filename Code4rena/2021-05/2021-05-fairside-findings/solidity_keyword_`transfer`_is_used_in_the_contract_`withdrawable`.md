## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Solidity keyword `transfer` is used in the contract `Withdrawable`](https://github.com/code-423n4/2021-05-fairside-findings/issues/67) 

# Handle

shw


# Vulnerability details

## Impact

The function `withdraw` in the contract `Withdrawable` uses the Solidity keyword, `transfer`, which is unrecommended since it forwards a fixed amount of 2300 gas to the recipient. The gas cost of opcodes may change during hard forks in the future and thus break the functionalities of existing deployed contracts.

## Proof of Concept

Referenced code:
[Withdrawable.sol#L18](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/dependencies/Withdrawable.sol#L18)

Please refer to the following references for more details:

[Solidity issue - Remove .send and .transfer](https://github.com/ethereum/solidity/issues/7455)
[Stop Using Solidity's transfer() Now](https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/)


## Recommended Mitigation Steps

Use `.call{value: 1 ether}("")` instead of `transfer` or `send`. Besides, since the `call` function forwards all gas to the recipient, the contract should add protections (e.g., reentrancy guards) to prevent the recipient from reentering critical functions.

