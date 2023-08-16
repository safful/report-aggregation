## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [SafeMath library is not always used in `Gravity`](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/60) 

# Handle

shw


# Vulnerability details

## Impact

SafeMath library functions are not always used in the `Gravity` contract's arithmetic operations, which could cause integer underflow/overflows. Using SafeMath is considered a best practice that could completely prevent underflow/overflows and increase code consistency.

## Proof of Concept

Referenced code:
[Gravity.sol#L202](https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/solidity/contracts/Gravity.sol#L202)
[Gravity.sol#L586](https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/solidity/contracts/Gravity.sol#L586)

## Recommended Mitigation Steps

Consider using the SafeMath library functions in the referenced lines of code.

