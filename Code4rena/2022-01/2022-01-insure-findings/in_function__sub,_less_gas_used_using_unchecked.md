## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [in function _sub, less gas used using unchecked](https://github.com/code-423n4/2022-01-insure-findings/issues/108) 

# Handle

Tomio


# Vulnerability details

## Impact
by using 'unchecked' you can save  +-182 gas

## Proof of Concept
before: https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L938
//22378 before

after:
```
function _sub(uint256 a, uint256 b) public pure returns (uint256) {
        if (a < b) {
            return 0;
        } else {
            unchecked {return a - b;}
        }
    }  
```
//22196 after


## Tools Used
Remix

## Recommended Mitigation Steps
used 'unchecked' in function _sub

