## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Overflow in _repay()](https://github.com/code-423n4/2021-12-sublime-findings/issues/58) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function repayPrincipal() calls _repay() with MAX_INT as parameter.
In _repay() this value (_amount) is multiplied by 10**30.
As _amount already has the maximum value of an int256 it will overflow.
Because solidity 7.6.0 is used and mul() isn't used (!) this actually works.
The resulting value is still large and thus the function repayPrincipal() does still work.

It is not recommended to rely on overflow working and when moving to solidity 0.8.x this will no longer work.

## Proof of Concept
https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/Pool/Repayments.sol#L23

```JS
    uint256 constant MAX_INT = 2**256 - 1;
```

https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/Pool/Repayments.sol#L377-L425

```JS
    
function repayPrincipal(address payable _poolID) external payable nonReentrant isPoolInitialized(_poolID) {
        ....
        uint256 _interestToRepay = _repay(_poolID, MAX_INT, true);
        ...

 function _repay( ... uint256 _amount,..) internal returns (uint256) {
..
        _amount = _amount * 10**30;
   
```

## Tools Used

## Recommended Mitigation Steps
Use safemath in _replay() and change MAX_INT to something like:
```JS
uint256 constant LARGE_INT = 2**128
```
Note: 10**30 ~ 2**100

