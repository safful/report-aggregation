## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: Conditional flow optimization in `Exchange.sol:removeLiquidity()`](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/28) 

# Handle

Dravee


# Vulnerability details

## Impact  
It's possible to save gas by optimizing conditional flows to avoid some unnecessary opcodes
  
## Proof of Concept  
In `Exchange.sol:removeLiquidity()`, the code is as follows:
```
File: Exchange.sol
225:         if (quoteTokenQtyToReturn > internalBalances.quoteTokenReserveQty) {
226:             internalBalances.quoteTokenReserveQty = 0;
227:         } else {
228:             internalBalances.quoteTokenReserveQty -= quoteTokenQtyToReturn;
229:         }
``` 

However, this can be optimized :  
- Strict inequalities (`>`) are more expensive than non-strict ones (`>=`). This is due to some supplementary checks (ISZERO, 3 gas)  
- In this case here, if `quoteTokenQtyToReturn == internalBalances.quoteTokenReserveQty`: `internalBalances.quoteTokenReserveQty = 0` should be used  
- Avoiding the else clause would avoid some opcodes (1 SUB = 3 gas, 2 MLOADs = 6 gas...)  
  
The code would become:  
```
File: Exchange.sol
225:         if (quoteTokenQtyToReturn >= internalBalances.quoteTokenReserveQty) {
226:             internalBalances.quoteTokenReserveQty = 0;
227:         } else {
228:             internalBalances.quoteTokenReserveQty -= quoteTokenQtyToReturn;
229:         }
```  
  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Use the non-strict greater-than operator in this particular case


