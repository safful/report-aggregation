## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Possible divide by zero errors in `Utils`](https://github.com/code-423n4/2021-07-spartan-findings/issues/232) 

# Handle

shw


# Vulnerability details

## Impact

Several functions in `Utils` do not handle edge cases where the divisor is 0, caused mainly by no liquidity in the pool. In such cases, the transactions revert without returning a proper error message.

## Proof of Concept

Referenced code:
[Utils.sol#L75](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L75)
[Utils.sol#L90](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L90)
[Utils.sol#L109-L110](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L109-L110)
[Utils.sol#L123-L124](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L123-L124)
[Utils.sol#L131](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L131)
[Utils.sol#L138](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L138)
[Utils.sol#L155](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L155)
[Utils.sol#L189](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L189)
[Utils.sol#L195](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L195)
[Utils.sol#L215](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Utils.sol#L215)

## Recommended Mitigation Steps

Check if the divisors are 0 in the above functions to handle edge cases.

