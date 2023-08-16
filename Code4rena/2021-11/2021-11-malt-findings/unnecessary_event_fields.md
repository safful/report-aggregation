## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary event fields](https://github.com/code-423n4/2021-11-malt-findings/issues/5) 

# Handle

TomFrench


# Vulnerability details

## Impact

Greater costs of epoch advancement

## Proof of Concept

The `Advance` event emits the block number and timestamp in its data

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/DAO.sol#L24

These fields are attached to events by default so it's unnecessary to manually emit them (and pay the associated gas costs)

## Recommended Mitigation Steps

Remove `block` and `timestamp` fields from `Advance` event

