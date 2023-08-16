## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Refacotr: Reuse same code for hashVariantTransactionData with txData and when preparedBlockNumber is 0](https://github.com/code-423n4/2021-07-connext-findings/issues/56) 

# Handle

GalloDaSballo


# Vulnerability details

## Impact

The code uses `hashVariantTransactionData` to verify the hash of the VariantTransactionData
It also uses
```
    variantTransactionData[digest] = keccak256(abi.encode(VariantTransactionData({
      amount: txData.amount,
      expiry: txData.expiry,
      preparedBlockNumber: 0
    })));
```

To generate VariantTransactionData with `preparedBlockNumber` set to 0

A simple refactoring of:
```
  function hashVariantTransactionData(TransactionData calldata txData) internal pure returns (bytes32) {
    return hashVariantTransaction(txData.amount, txData.expiry, txData.preparedBlockNumber)
  }

  function hashVariantTransaction(uint256 amount, uint256 expiry, uint256 prepareBlocNumber) internal pure returns (bytes32) {
    return keccak256(abi.encode(VariantTransactionData({
      amount: amount,
      expiry: expiry,
      preparedBlockNumber: preparedBlockNumber
    })));
  }

```

This would allow to further steamline the code from
```
    variantTransactionData[digest] = keccak256(abi.encode(VariantTransactionData({
      amount: txData.amount,
      expiry: txData.expiry,
      preparedBlockNumber: 0
    })));
```

to 
```
    variantTransactionData[digest] = hashVariantTransaction(txData.amount, txData.expiry, 0)
```

## Recommended Mitigation Steps
This has no particular benefit beside making all code related to Variant Data consistent 

