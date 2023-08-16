## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- sponsor disputed
- disagree with severity

# [Prevent underflow in require](https://github.com/code-423n4/2021-09-swivel-findings/issues/62) 

# Handle

gpersoon


# Vulnerability details

## Impact
The contract Swivel.sol contains a few of the following requires:
 require(a <= (o.premium - filled[hash]), 'taker amount > available volume');  

If "o.premium" happens to be smaller than "filled[hash]", a revert will occur at "o.premium - filled[hash]" and no error message will be displayed.

Also note the statements use slightly different syntax with the parentheses. 

## Proof of Concept
Swivel.sol:    require(a <= (o.premium    - filled[hash]),  'taker amount > available volume');
Swivel.sol:    require((a <= o.principal  - filled[hash]),  'taker amount > available volume');  // slightly different syntax
Swivel.sol:    require(a <= ((o.principal - filled[hash])), 'taker amount > available volume');  // slightly different syntax
Swivel.sol:    require(a <= (o.premium    - filled[hash]),  'taker amount > available volume');
Swivel.sol:    require(a <= (o.premium    - filled[hash]),  'taker amount > available volume');
Swivel.sol:    require(a <= (o.principal  - filled[hash]),  'taker amount > available volume');
Swivel.sol:    require(a <= (o.principal  - filled[hash]),  'taker amount > available volume');
Swivel.sol:    require(a <= (o.premium    - filled[hash]),  'taker amount > available volume');

## Tools Used

## Recommended Mitigation Steps
replace
 require(a <= (o.xxxx - filled[hash]), 'taker amount > available volume');  
with
 require( (a + filled[hash]) <= o.xxxx), 'taker amount > available volume');

Use the same parentheses structure everywhere.

