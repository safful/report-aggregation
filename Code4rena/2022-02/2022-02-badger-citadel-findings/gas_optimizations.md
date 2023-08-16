## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-02-badger-citadel-findings/issues/9) 

# Handle

PostMan56


# Vulnerability details

## Impact
This is my first time submitting so please let me know if it's bad or poorly explained 

Small gas optimization in the function  'getTokenInLimitLeft' and use of uint256 vs uint8

## Proof of Concept 'getTokenInLimitLeft'
    function getTokenInLimitLeft() external view returns (uint256 limitLeft_) {
        if (totalTokenIn < tokenInLimit) {
            limitLeft_ = tokenInLimit - totalTokenIn;
        }
    }
In gas estimates(before):
```
"Creation": {   
    "codeDepositCost": "3427600",
    "executionCost": "3828",
    "totalCost": "3431428"
}
```
    function getTokenInLimitLeft() external view returns (uint256 limitLeft_) {
        if (totalTokenIn <= tokenInLimit) {
            limitLeft_ = tokenInLimit - totalTokenIn;
        }
    }

In gas estimates(after):
```
"Creation": {
    "codeDepositCost": "3427400",
    "executionCost": "3828",
    "totalCost": "3431228"
}
```

## Proof of Concept uint256 vs uint8
uint8 can be found in lines:
56,
57,
65,
146

In gas estimates(before):
```
"Creation": {
    "codeDepositCost": "3427600",
    "executionCost": "3828",
    "totalCost": "3431428"
}
```
In gas estimates(after):
```
"Creation": {
    "codeDepositCost": "3394800",
    "executionCost": "3787",
    "totalCost": "3398587"
}
```
## Tools Used
Remix gas estimates

## Recommended Mitigation Steps
< & > contains ISZERO opcode making it cost more
To negate this add = after < or > to save 200 gas on contract deploy

use of uint256 is cheaper than uint8 in data types

