## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [absolute difference is not calculated properly when a > b in MathUtils](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/139) 

# Handle

hack3r-0m


# Vulnerability details

https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/libraries/MathUtils.sol#L22
the difference is computed incorrectly when a > b.

As it only used in within1 function, scope narrows down to where `difference(a, b) <= 1;` is exploitable.

cases where `difference(a, b) <= 1` should be true but is reported false:
- where b = a-1 (returned value is type(uint256).max)

cases where `difference(a, b) <= 1` should be false but is reported true:
- where a = type(uint256).max and b = 0, it returns 1 but it should ideally return type(uint256).max

within1 is used at the following locations:
- https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/pool/HybridPool.sol#L359
- https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/pool/HybridPool.sol#L383
- https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/pool/HybridPool.sol#L413

It is possible to decrease the denominator and increase the value of the numerator (when calculating y) using constants and input to make within1 fail

Mitigation:

Add `else` condition to mitigate it.

```
unchecked {
          if (a > b) {
              diff = a - b;
          }
          else {
              diff = b - a;   
          }
      }
```

(re-submitting this issue after withdrawing past one since I forgot to add more  details and POC)

