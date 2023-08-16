## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Avoid expensive calculation by checking if `valueAll() == 0`](https://github.com/code-423n4/2022-01-insure-findings/issues/344) 

# Handle

Dravee


# Vulnerability details

## Impact
Checking if the value is 0 before returning 0 is less expensive than returning a calculation that's equal to 0

## Proof of Concept
In `Vault.sol:underlyingValue()`, the code is as follows:
```
Vault.sol
400:     function underlyingValue(address _target)
401:         public
402:         view
403:         override
404:         returns (uint256)
405:     {
406:         if (attributions[_target] > 0) {
407:             return (valueAll() * attributions[_target]) / totalAttributions;
408:         } else {
409:             return 0;
410:         }
411:     }
```
It can be optimized as such: 
```
406:         uint256 valueAll = valueAll();
407:         if (valueAll != 0 && attributions[_target] > 0) {
408:             return (valueAll * attributions[_target]) / totalAttributions;
409:         } else {
410:             return 0;
411:         }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Cache the loaded storage value in a memory variable and make the 0 checks to avoid unnecessary calculations if `valueAll() == 0`

