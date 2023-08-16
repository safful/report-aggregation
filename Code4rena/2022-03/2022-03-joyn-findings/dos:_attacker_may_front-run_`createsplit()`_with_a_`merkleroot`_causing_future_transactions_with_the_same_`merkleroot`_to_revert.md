## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [DoS: Attacker May Front-Run `createSplit()` With A `merkleRoot` Causing Future Transactions With The Same `merkleRoot` to Revert](https://github.com/code-423n4/2022-03-joyn-findings/issues/33) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/SplitFactory.sol#L153-L155


# Vulnerability details

## Impact

A `merkleRoot` may only be used once in `createSplit()` since it is used as `salt` to the deployment of a `SplitProxy`. 

The result is an attacker may front-run any `createSplit()` transaction in the mem pool and create another `createSplit()` transaction with a higher gas price that uses the same `merkleRoot` but changes the other fields such as the `_collectionContract` or `_splitAsset()`.  The original transaction will revert and the user will not be able to send any more transaction with this `merkleRoot`.

The user would therefore have to generate a new merkle tree with different address, different allocations or a different order of leaves in the tree to create a new merkle root. However, the attack is repeateable and there is no guarantee this new merkle root will be successfully added to a split without the attacker front-running the transaction again.

## Proof of Concept

The excerpt from `createSplitProxy()` shows the `merkleRoot()` being used as a `salt`.

```
  splitProxy = address(
    new SplitProxy{salt: keccak256(abi.encode(merkleRoot))}()
  );
```

## Recommended Mitigation Steps

As seems to be the case here if the transaction address does NOT need to be known ahead of time consider removing the `salt` parameter from the contract deployment.

Otherwise, if the transaction address does need to be known ahead of time then consider concatenating `msg.sender` to the `merkleRoot`. e.g.

```
splitProxy = address(
    new SplitProxy{salt: keccak256(abi.encode(msg.sender, merkleRoot))}()
  )
```

