## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Using access lists can save gas due to EIP-2930 post-Berlin hard fork](https://github.com/code-423n4/2021-07-connext-findings/issues/39) 

# Handle

0xRajeev


# Vulnerability details

## Impact

EIP-2929 in Berlin fork increased the gas costs of SLOADs and CALL* family opcodes increasing them for not-accessed slots/addresses and decreasing them for accessed slots. EIP-2930 optionally supports specifying an access list (in the transaction) of all slots and addresses accessed by the transaction which reduces their gas cost upon access and prevents EIP-2929 gas cost increases from breaking contracts. 

Impact: Considering these changes may significantly impact gas usage for transactions that call functions touching many state variables or making many external calls. Specifically, removeUserActiveBlocks() removes an active block from the array of blocks for an user, all of which are stored in storage. Transactions for fulfill() and cancel() functions that call removeUserActiveBlocks()  can consider using access lists for all the storage state (of user’s active blocks) they touch (read + write) to reduce gas.


## Proof of Concept

https://eips.ethereum.org/EIPS/eip-2929

https://eips.ethereum.org/EIPS/eip-2930

https://hackmd.io/@fvictorio/gas-costs-after-berlin

https://github.com/gakonst/ethers-rs/issues/265

SLOADs: https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L580

SSTOREs: https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L583

Calls: https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L346

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L490


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Evaluate the feasibility of using access lists to save gas due to EIPs 2929 & 2930 post-Berlin hard fork. The tooling support is WIP.

