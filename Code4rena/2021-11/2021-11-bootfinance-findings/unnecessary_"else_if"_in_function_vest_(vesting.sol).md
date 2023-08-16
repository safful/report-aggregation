## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary "else if" in function vest (Vesting.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/22) 

# Handle

ye0lde


# Vulnerability details

## Impact

The "else if(_isRevocable == 1)" is not needed and can be removed to save gas and improve code clarity.

## Proof of Concept

The "_isRevocable" variable is guaranteed to be 0 or 1 here:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/Vesting.sol#L77

But it is treated like it can have some other value here:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/Vesting.sol#L83-L88

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Rewrite these lines
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/Vesting.sol#L83-L88
to
<code>
 benRevocable[_beneficiary] = (_isRevocable == 0) ? [false,false] : [true,false];
</code>

