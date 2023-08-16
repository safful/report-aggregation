## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Revert string > 32 bytes](https://github.com/code-423n4/2022-01-notional-findings/issues/110) 

# Handle

sirhashalot


# Vulnerability details

## Impact

Strings are broken into 32 byte chunks for operations. Revert error strings over 32 bytes therefore consume extra gas as [documented publicly](https://blog.polymath.network/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size-c44580b218e6#c17b)

## Proof of Concept

There are multiple examples of this gas optimization opportunity, including but not limited to:
- TreasuryAction.sol [line 41](https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/TreasuryAction.sol#L41)

## Recommended Mitigation Steps

Reducing revert error strings to under 32 bytes decreases deployment time gas and runtime gas when the revert condition is met. Alternatively, the code could be modified to use custom errors, introduced in Solidity 0.8.4: https://blog.soliditylang.org/2021/04/21/custom-errors/

