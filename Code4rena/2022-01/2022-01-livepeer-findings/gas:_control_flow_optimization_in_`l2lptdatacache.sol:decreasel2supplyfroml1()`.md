## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Control flow optimization in `L2LPTDataCache.sol:decreaseL2SupplyFromL1()`](https://github.com/code-423n4/2022-01-livepeer-findings/issues/137) 

# Handle

Dravee


# Vulnerability details

## Impact
Increased gas cost

## Proof of Concept
In `L2LPTDataCache.sol:decreaseL2SupplyFromL1()`, the code is as follows: 
```
File: L2LPTDataCache.sol
57:     function decreaseL2SupplyFromL1(uint256 _amount) external onlyL2LPTGateway {
58:         // If there is a mass withdrawal from L2, _amount could exceed l2SupplyFromL1.
59:         // In this case, we just set l2SupplyFromL1 = 0 because there will be no more supply on L2
60:         // that is from L1 and the excess (_amount - l2SupplyFromL1) is inflationary LPT that was
61:         // never from L1 in the first place.
62:         if (_amount > l2SupplyFromL1) { 
63:             l2SupplyFromL1 = 0;
64:         } else {
65:             l2SupplyFromL1 -= _amount;
66:         }
67: 
68:         // No event because the L2LPTGateway events are sufficient
69:     }
```

However, this can be optimized :
- Strict inequalities (`>`) are more expensive than non-strict ones (`>=`). This is due to some supplementary checks (ISZERO)
- In this case here, if `_amount == l2SupplyFromL1`, `0` should be returned
- Avoiding the else clause would avoid some opcodes (1 SUB,  1 SLOAD, 1 MLOAD)

The code would become: 
```
        if (_amount >= l2SupplyFromL1) {
            l2SupplyFromL1 = 0;
        } else {
            l2SupplyFromL1 -= _amount;
        }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Use the non-strict greater-than operator in this particular case

