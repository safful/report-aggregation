## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoid repeating storage reads in a loop to save gas](https://github.com/code-423n4/2021-07-sherlock-findings/issues/149) 

# Handle

shw


# Vulnerability details

## Impact

A storage read cost more gas than a memory read. State variables that do not change during a loop can be stored in local variables and be read from memory multiple times to save gas.

## Proof of Concept

Referenced code:
[LibPool.sol#L89](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/libraries/LibPool.sol#L89)
[LibSherX.sol#L60](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/libraries/LibSherX.sol#L60)
[LibSherX.sol#L94](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/libraries/LibSherX.sol#L94)
[PoolBase.sol#L131](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/PoolBase.sol#L131)
[SherX.sol#L76](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherX.sol#L76)
[SherX.sol#L98](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherX.sol#L98)
[SherX.sol#L152](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherX.sol#L152)
[SherX.sol#L184](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherX.sol#L184)
[SherX.sol#L243](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherX.sol#L243)
[Gov.sol#L190](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/Gov.sol#L190)

## Recommended Mitigation Steps

For example, consider re-writing the `harvestFor(address)` function of `SherX` as follows:

```solidity
function harvestFor(address _user) public override {
  GovStorage.Base storage gs = GovStorage.gs();
  uint256 len = gs.tokensStaker.length;
  for (uint256 i; i < len; i++) {
    PoolStorage.Base storage ps = PoolStorage.ps(gs.tokensStaker[i]);
    harvestFor(_user, ps.lockToken);
  }
}
```

