## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Simpler logic can save gas](https://github.com/code-423n4/2021-06-gro-findings/issues/44) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The for loop in investSingle() can be removed in favor of simpler logic to calculate k [k = N_COINS - (i + j)], which will save some gas in the deposit flow.


## Proof of Concept
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/pools/LifeGuard3Pool.sol#L317-L326


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Replace L317 to L323 with:
```
uint256 inBalance = inAmounts[N_COINS - (i + j)];
if (inBalance > 0) {
      _exchange(inBalance, int128(k), int128(i));
}
```

