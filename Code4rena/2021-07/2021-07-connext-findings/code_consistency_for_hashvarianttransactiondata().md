## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Code Consistency for hashVariantTransactionData()](https://github.com/code-423n4/2021-07-connext-findings/issues/22) 

# Handle

greiart


# Vulnerability details

### Proof of Concept

`hashVariantTransactionData()` should follow the same style of `hashInvariantTransactionData()` and the recover signature functions, where the payload is generated is stored in memory before hashing. Preliminary tests in remix show that it is minimally more gas efficient as well.

```jsx
function hashVariantTransactionData(TransactionData calldata txData) internal pure returns (bytes32) {
    VariantTransactionData memory variant = VariantTransactionData({
      amount: txData.amount,
      expiry: txData.expiry,
      preparedBlockNumber: txData.preparedBlockNumber
		});
		return keccak256(abi.encode(variant));
  }
```

### Alternative View on Notion

[https://www.notion.so/Code-Consistency-for-hashVariantTransactionData-33bf6578a16c4b18896f4d7ca7582e21](https://www.notion.so/Code-Consistency-for-hashVariantTransactionData-33bf6578a16c4b18896f4d7ca7582e21)

