## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Possible DoS attack when creating `Joins` in `Wand`](https://github.com/code-423n4/2021-05-yield-findings/issues/70) 

# Handle

shw


# Vulnerability details

## Impact

It is possible for an attacker to intendedly create a fake `Join` corresponding to a specific token beforehand to make `Wand` unable to deploy the actual `Join`, causing a DoS attack.

## Proof of Concept

The address of `Join` corresponding to an underlying `asset` is determined as follows and thus unique:

```solidity
Join join = new Join{salt: keccak256(abi.encodePacked(asset))}();
```

Besides, the function `createJoin` in the contract `JoinFactory` is permissionless: Anyone can create the `Join` corresponding to the `asset`. An attacker could then deploy a large number of `Joins` with different common underlying assets (e.g., DAI, USDC, ETH) before the `Wand` deploying them. The attempt of deploying these `Joins` by `Wand` would fail since the attacker had occupied the desired addresses with fake `Joins`, resulting in a DoS attack.

Moreover, the attacker can also perform DoS attacks on newly added assets: He monitors the mempool to find transactions calling the function `addAsset` of `Wand` and front-runs them to create the corresponding `Join` to make the benign transaction fail.

Referenced code:
[JoinFactory.sol#L64-L75](https://github.com/code-423n4/2021-05-yield/blob/main/contracts/JoinFactory.sol#L64-L75)
[Wand.sol#L53](https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Wand.sol#L53)

## Recommended Mitigation Steps

Enable access control in `createJoin` (e.g., adding the `auth` modifier) and allow `Wand` to call it.

