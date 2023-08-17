## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-01

# [Incorrect Encoding of Order Hashes](https://github.com/code-423n4/2023-01-opensea-findings/issues/61) 

# Lines of code

https://github.com/ProjectOpenSea/seaport/blob/5de7302bc773d9821ba4759e47fc981680911ea0/contracts/lib/ConsiderationEncoder.sol#L569-L574


# Vulnerability details

## Impact

The order hashes are incorrectly encoded during the `_encodeOrderHashes` mechanism, causing functions such as `_encodeRatifyOrder` and `_encodeValidateOrder` to misbehave.

## Proof of Concept

The order hashes encoding mechanism appears to be incorrect as the instructions `srcLength.next().offset(headAndTailSize)` will cause the pointer to move to the end of the array (i.e. `next()` skips the array's `length` bitwise entry and `offset(headAndTailSize)` causes the pointer to point right after the last element). In turn, this will cause the `0x04` precompile within `MemoryPointerLib::copy` to handle the data incorrectly and attempt to copy data from the `srcLength.next().offset(headAndTailSize)` pointer onwards which will be un-allocated space and thus lead to incorrect bytes being copied.

## Tools Used

Manual inspection of the codebase, documentation of the ETH precompiles, and the Solidity compiler documentation.

## Recommended Mitigation Steps

We advise the `offset` instruction to be omitted as the current implementation will copy from unsafe memory space, causing data corruption in the worst-case scenario and incorrect order hashes being specified in the encoded payload. As an additional point, the `_encodeOrderHashes` will fail execution if the array of order hashes is empty as a `headAndTailSize` of `0` will cause the `MemoryPointerLib::copy` function to fail as the precompile would yield a `returndatasize()` of `0`.