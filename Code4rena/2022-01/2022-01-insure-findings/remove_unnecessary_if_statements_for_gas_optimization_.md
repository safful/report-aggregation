## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Remove unnecessary if statements for gas optimization ](https://github.com/code-423n4/2022-01-insure-findings/issues/186) 

# Handle

ospwner


# Vulnerability details

## Impact
Checking arrays' length before using it in a for loop is unnecessary when array's length is used in loop exit condition. 

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Factory.sol#L175

```
        if (_references.length > 0) {
            for (uint256 i = 0; i < _references.length; i++) 
```

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Factory.sol#L185
```
        if (_conditions.length > 0) {
            for (uint256 i = 0; i < _conditions.length; i++) 
```


## Recommended Mitigation Steps

Remove the two unnecessary  if statements.


