## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache storage variables to local variables to save gas](https://github.com/code-423n4/2021-07-connext-findings/issues/75) 

# Handle

shw


# Vulnerability details

## Impact

In general, if a state variable is read more than once, caching its value to a local variable and reusing it will save gas since a storage read spends more gas than a memory write plus a memory read.

## Proof of Concept

Referenced code:
[TransactionManager.sol#L122-L125](https://github.com/code-423n4/2021-07-connext/blob/main/contracts/TransactionManager.sol#L122-L125)
[TransactionManager.sol#L254-L260](https://github.com/code-423n4/2021-07-connext/blob/main/contracts/TransactionManager.sol#L254-L260)

## Recommended Mitigation Steps

Rewrite #L122-L125 as follows:

```solidity
uint256 balance = routerBalances[msg.sender][assetId];
require(balance >= amount, "removeLiquidity: INSUFFICIENT_FUNDS");

// Update router balances
routerBalances[msg.sender][assetId] = balance - amount;
```

Rewrite #L254-L260 as follows:

```solidity
uint256 balance = routerBalances[invariantData.router][invariantData.receivingAssetId];
require(
    balance >= amount,
    "prepare: INSUFFICIENT_LIQUIDITY"
);

// Decrement the router liquidity
routerBalances[invariantData.router][invariantData.receivingAssetId] = balance - amount;
```

