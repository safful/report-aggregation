## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [The `domainSeperator` is not recalculated after a hard fork happens](https://github.com/code-423n4/2021-06-realitycards-findings/issues/166) 

# Handle

shw


# Vulnerability details

## Impact

The variable `domainSeperator` in `EIP712Base` is cached in the contract storage and will not change after the contract is initialized. However, if a hard fork happens after the contract deployment, the `domainSeperator` would become invalid on one of the forked chains due to the `block.chainid` has changed.

## Proof of Concept

Referenced code:
[EIP712Base.sol#L25-L44](https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/lib/EIP712Base.sol#L25-L44)

## Recommended Mitigation Steps

Consider using the [implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/draft-EIP712.sol) from OpenZeppelin, which recalculates the domain separator if the current `block.chainid` is not the cached chain ID.

