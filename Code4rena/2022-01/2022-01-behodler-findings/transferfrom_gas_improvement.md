## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [transferFrom gas improvement](https://github.com/code-423n4/2022-01-behodler-findings/issues/187) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The ERC20Burnable.sol file has code copied from the OpenZeppelin ERC20.sol contract. The Behodler code `transferFrom()` function does use the latest version of the OpenZeppelin code, modified earlier in Jan 2022 in [PR 3085](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3085), which can save gas if currentAllowance == type(uint256).max.

A second gas savings that has been present in OpenZeppelin for some time but is not in the Behodler code is to add an unchecked clause around the `approve()` call.

## Proof of Concept

The Behodler `transferFrom()` function [doesn't use the latest edits from OZ or the unchecked clause on the approve call](https://github.com/code-423n4/2022-01-behodler/blob/cedb81273f6daf2ee39ec765eef5ba74f21b2c6e/contracts/ERC677/ERC20Burnable.sol#L204-L218). In contrast, the OZ code [does use these edits for gas savings](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4f8af2dceb0fbc36cb32eb2cc14f80c340b9022e/contracts/token/ERC20/ERC20.sol#L156-L172).

## Recommended Mitigation Steps

Use the latest OZ edits and the unchecked clause for gas savings if it doesn't introduce overflow or underflow conditions.

