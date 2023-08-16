## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# ["constants" expressions are expressions, not constants. Use "immutable" instead.](https://github.com/code-423n4/2022-01-timeswap-findings/issues/62) 

# Handle

Dravee


# Vulnerability details

## Impact  
Due to how `constant` variables are implemented, an expression assigned to a `constant` variable is recomputed each time that the variable is used, which wastes some gas.  
  
If the variable was `immutable` instead: the calculation would only be done once at deploy time (in the constructor), and then the result would be saved and read directly at runtime rather than being recalculated.
  
See: [ethereum/solidity#9232]([https://github.com/ethereum/solidity/issues/9232](https://github.com/ethereum/solidity/issues/9232))  
  
## Proof of Concept  
```  
Timeswap-V1-Convenience\contracts\libraries\DateTime.sol:31:    uint constant SECONDS_PER_DAY = 24 * 60 * 60;
Timeswap-V1-Convenience\contracts\libraries\DateTime.sol:32:    uint constant SECONDS_PER_HOUR = 60 * 60;
```  
  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Change these expressions from `constant` to `immutable` and implement the calculation in the constructor


