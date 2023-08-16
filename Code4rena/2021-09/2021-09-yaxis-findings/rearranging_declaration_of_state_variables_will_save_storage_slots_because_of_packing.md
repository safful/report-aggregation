## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Rearranging declaration of state variables will save storage slots because of packing](https://github.com/code-423n4/2021-09-yaxis-findings/issues/43) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Storage slots are allocated based on the declaration order of state variables in contract definitions. For types less than 256 bits, they can be packed by the compiler if more than one fit into the same 32B storage slot. This reduces the number of storage slots but may increase runtime gas consumption because of masking the other shared variables in slot. However, if variables used together in function logic are packed in the same slot, it allows the compiler to optimize SLOADs/SSTOREs.

Example: An example of this is the declaration of the halted boolean state variable. Given the current declaration order, this occupies a full slot because booleans are internally represented by uint8 and the neighbouring declarations are uint256 which need a full slot for themselves.

Moving the halted bool next to governance address variable declaration will allow those two to share a slot. This reduces one slot and also should not incur extra masking gas overhead at runtime because governor and halted are used in onlyGovernance and notHalted modifiers respectively which are typically used together.

## Proof of Concept

https://github.com/code-423n4/2021-09-yaxis/blob/cf7d9448e70b5c1163a1773adb4709d9d6ad6c99/contracts/v3/Manager.sol#L33-L49


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Move halted declaration to immediately after governance declaration. Also, consider the declaration order of all state variables across contracts for such packing possibilities.

