## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Incorrect naming issue](https://github.com/code-423n4/2021-11-overlay-findings/issues/26) 

# Handle

xYrYuYx


# Vulnerability details

## Impact
https://github.com/code-423n4/2021-11-overlay/blob/main/contracts/OverlayV1UniswapV3Market.sol#L171

_tickToPrice function has underscore even it is public function.
Underscore is used to indicate internal or private functions.


## Tools Used
Manual

## Recommended Mitigation Steps
Change function to internal or private, or remove underscore if you want to keep it as public function.

