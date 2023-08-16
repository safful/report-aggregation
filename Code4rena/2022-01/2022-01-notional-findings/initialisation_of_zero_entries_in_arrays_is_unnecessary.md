## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Initialisation of zero entries in arrays is unnecessary](https://github.com/code-423n4/2022-01-notional-findings/issues/59) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
Gas costs.
## Proof of Concept

In a number of places we create an array and then fill every element with zero. There's no need to do this as a newly declared array will have zero-valued elements by default. We can then avoid the costs of writing a new zero to them.

For example we could remove these three lines entirely:
https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L111-L113

We can then just pass an empty array in the lines below as so.
```
BALANCER_VAULT.exitPool(
  NOTE_ETH_POOL_ID,
  address(this),
  payable(owner), // Owner will receive the NOTE and WETH
  IVault.ExitPoolRequest(
      assets,
      new uint256[](2), // inlined here
      abi.encode(
          IVault.ExitKind.EXACT_BPT_IN_FOR_TOKENS_OUT,
          bptExitAmount
      ),
      false // Don't use internal balances
  )
);
```

This also crops up elsewhere:
https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L156
https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L169
https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L185

## Tools Used

## Recommended Mitigation Steps

Omit lines writing zeros to an empty array.

