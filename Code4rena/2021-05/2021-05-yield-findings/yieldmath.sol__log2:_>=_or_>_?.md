## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [YieldMath.sol / Log2: >= or > ?](https://github.com/code-423n4/2021-05-yield-findings/issues/2) 

# Handle

gpersoon


# Vulnerability details

## Impact
The V1 version of YieldMath.sol contains ">=" (larger or equal), while the V2 version of YieldMath.sol containt ">" (larger) in the log_2 function.
This change doesn't seem logical and might lead to miss calculations.
The difference is present in a number of adjacent lines.

## Proof of Concept
// https://github.com/yieldprotocol/yieldspace-v1/blob/master/contracts/YieldMath.sol#L217
function log_2 (uint128 x)
...
b = b * b >> 127; if (b >= 0x100000000000000000000000000000000) {b >>= 1; l |= 0x1000000000000000000000000000000;}


//https://github.com/code-423n4/2021-05-yield/blob/main/contracts/yieldspace/YieldMath.sol#L58
function log_2(uint128 x)
...
b = b * b >> 127; if(b > 0x100000000000000000000000000000000) {b >>= 1; l |= 0x1000000000000000000000000000000;}

## Tools Used
diff

## Recommended Mitigation Steps
Check which version is the correct version and fix the incorrect version.



