## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- resolved
- sponsor confirmed

# [Gas: Unused Named Returns](https://github.com/code-423n4/2022-01-yield-findings/issues/60) 

# Handle

Dravee


# Vulnerability details

## Impact  
Using both named returns and a return statement isn't necessary.
Removing unused named return variables can reduce gas usage and improve code clarity. To save gas and improve code quality: consider using only one of those.  
  
## Proof of Concept  
Instances include:  
```
ConvexStakingWrapper.sol:310:    function earned(address _account) external view returns (EarnedData[] memory claimable) { //@audit-info 342: return claimable;

Cvx3CrvOracle.sol:76:        returns (uint256 quoteAmount, uint256 updateTime) //@audit-info 78: return _peek(base.b6(), quote.b6(), baseAmount);

Cvx3CrvOracle.sol:97:        returns (uint256 quoteAmount, uint256 updateTime) //@audit-info 99: return _peek(base.b6(), quote.b6(), baseAmount);
``` 
  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Remove the unused named returns


