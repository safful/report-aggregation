## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [unnecessary checked postfix arithmetics](https://github.com/code-423n4/2022-01-insure-findings/issues/76) 

# Handle

egjlmn1


# Vulnerability details

in all of your for loops, you increase your loop variable using `i++`
it has 2 problems:
1. postfix increment is more wasteful than prefix increment (`++i` instead of `i++`)
2. there is no risk for overflow, so you can use `unchecked{}`

## Impact
prefix arithmetic is a bit cheaper than postfix arithmetic, but if you do it in a for loop, this small amount of gas can pile up and be a big waste.
also, in solidity 0.8.0+, every arithmetic operation is checked for overflow and underflow, which adds a lot of gas to a single operation. Since in your for loop you don't have the risk for overflow, you can surround the operation in `unchecked{}` to save a lot of gas (which will save a huge amount since it saves a lot in a single loop iteration.)

## Proof of Concept
Checked on remix

## Tools Used
manual code review

## Recommended Mitigation Steps
change every `i++` in your for loops to `unchecked{++i}`

