## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Reduce rounding error when minting UDT in Unlock](https://github.com/code-423n4/2021-11-unlock-findings/issues/131) 

# Handle

HardlyDifficult


# Vulnerability details

## Impact
`maxTokens` in Unlock's `recordKeyPurchase` currently rounds more than is required.

## Proof of Concept
Plug the formula in Wolfgram Alpha to simplify from:

```
            maxTokens = IMintableERC20(udt).balanceOf(address(this)) * valueInETH / (2 + 2 * valueInETH / grossNetworkProduct) / grossNetworkProduct;
```

to
```
            maxTokens = IMintableERC20(udt).balanceOf(address(this)) * valueInETH / (2 * (valueInETH + grossNetworkProduct));
```

Example inputs:
```
balance: 10000
price: 0.012345678912345678
gnp: 1000 + 0.012345678912345678 (for this purchase)

61728394561728390 old formula
61727632492197622 new formula (smaller than old)
61726870441482920.98 actual per wolfgram (smaller than new)
1524120245470 delta old - actual
762050714702 delta new - actual
```

The "new" formula proposed above is closer to the expected value. It's also easier to read and saves 123 gas.

## Tools Used
https://www.wolframalpha.com/

## Recommended Mitigation Steps

