## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Duplicate LP token could lead to incorrect deposits](https://github.com/code-423n4/2022-05-vetoken-findings/issues/11) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/Booster.sol#L256


# Vulnerability details

## Impact
It was observed that addPool function is not checking for duplicate lpToken which allows 2 or more pools to have exact same lpToken. This can cause issue with deposits.

In case of duplicate lpToken, the first pool calling depositAll will take away all lpToken and deposit them under there own pid. This leaves no balance for 2nd pool

## Proof of Concept

1. PoolManager call addPool function and uses lpToken as A
2. PoolManager again call addPool function and mistakenly provides lpToken as A
3. Now 2 pools will be created with lpToken as A
4. depositAll function is called passing first pool. 
5. This takes all balance of lpToken A and depsoit it under first pool pid
6. This mean no balance is left for second pool now

## Recommended Mitigation Steps
Add a global variable keeping track of all lpToken added for pool. In case of duplicate lpToken addPool function should fail.

