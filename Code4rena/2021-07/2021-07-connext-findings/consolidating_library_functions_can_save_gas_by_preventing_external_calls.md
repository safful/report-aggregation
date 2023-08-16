## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Consolidating library functions can save gas by preventing external calls](https://github.com/code-423n4/2021-07-connext-findings/issues/42) 

# Handle

0xRajeev


# Vulnerability details

## Impact

While code modularity is generally a good practice and creating libraries of functions commonly used across different contracts can increase maintainability and reduce contract deployment size/cost, it comes at the increased cost of gas usage at runtime because of the external calls. EIP-2929 in Berlin fork increased the gas costs of CALL* family opcodes to 2600. Making a delegatecall to a library function therefore costs 2600. 

Impact: A LibAsset.transferAsset() call from TransactionManager.sol makes LibERC20.transfer() call for ERC20 which in turn makes another external call to LibUtils.revertIfCallFailed() in wrapCall. So an ERC20 transfer effectively makes 3 additional (besides the ERC20 token contract function call assetId.call(..) external calls -> LibAsset -> LibERC20 -> LibUtils, which costs 2600*3 = 7800 gas.
 
Combining these functions into a single library or making them all internal to TransactionManager.sol can convert these delegatecalls into JMPs to save gas.

## Proof of Concept

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/lib/LibAsset.sol#L58

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/lib/LibAsset.sol#L44

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/lib/LibERC20.sol#L64

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/lib/LibERC20.sol#L20

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L128

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L369

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L378

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L406

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L425

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L504

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L514

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L525

And other Lib* calls.

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Consider moving all the library functions internal to this contract or to a single library to save gas from external calls each of which costs 2600 gas.

