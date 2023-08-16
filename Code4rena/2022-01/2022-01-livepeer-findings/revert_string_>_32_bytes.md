## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Revert string > 32 bytes](https://github.com/code-423n4/2022-01-livepeer-findings/issues/115) 

# Handle

sirhashalot


# Vulnerability details

## Impact

Strings are broken into 32 byte chunks for operations. Revert error strings over 32 bytes therefore consume extra gas than shorter strings, as [documented publicly](https://blog.polymath.network/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size-c44580b218e6#c17b).

## Proof of Concept

There are dozens of examples of this gas optimization opportunity in the project, but some examples of this issue include:
- [L2/gateway/L2Migrator.sol:184](https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2Migrator.sol#L184)
- [L2/gateway/L2Migrator.sol:201](https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2Migrator.sol#L201)
- [L2/gateway/L2Migrator.sol:221](https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2Migrator.sol#L221)
- [L2/gateway/L2Migrator.sol:257](https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2Migrator.sol#L257)
- [bonding/BondingManager.sol:553](https://github.com/livepeer/protocol/blob/20e7ebb86cdb4fe9285bf5fea02eb603e5d48805/contracts/bonding/BondingManager.sol#L553)
- [bonding/BondingManager.sol:463](https://github.com/livepeer/protocol/blob/20e7ebb86cdb4fe9285bf5fea02eb603e5d48805/contracts/bonding/BondingManager.sol#L463)
- [bonding/BondingManager.sol:651](https://github.com/livepeer/protocol/blob/20e7ebb86cdb4fe9285bf5fea02eb603e5d48805/contracts/bonding/BondingManager.sol#L651)
- [token/Minter.sol:78](https://github.com/livepeer/protocol/blob/20e7ebb86cdb4fe9285bf5fea02eb603e5d48805/contracts/token/Minter.sol#L78)
- [token/Minter.sol:80](https://github.com/livepeer/protocol/blob/20e7ebb86cdb4fe9285bf5fea02eb603e5d48805/contracts/token/Minter.sol#L80)
- [token/Minter.sol:93](https://github.com/livepeer/protocol/blob/20e7ebb86cdb4fe9285bf5fea02eb603e5d48805/contracts/token/Minter.sol#L93)

## Recommended Mitigation Steps

Reducing revert error strings to under 32 bytes decreases deployment time gas and runtime gas when the revert condition is met. Alternatively, use custom errors, introduced in Solidity 0.8.4: https://blog.soliditylang.org/2021/04/21/custom-errors/

