## Tags

- bug
- sponsor confirmed
- disagree with severity
- 0 (Non-critical)
- resolved

# [No zero address check for controller in TempusPool](https://github.com/code-423n4/2021-10-tempus-findings/issues/6) 

# Handle

loop


# Vulnerability details

TempusPool needs to be initialized with a valid and existing controller. When initializing a pool `address controller` is passed to the constructor of a pool implementation. This `address` is then passed as `address ctrl` to the TempusPool constructor where it is set to the immutable `address controller`. If a pool accidentally gets initialized with the zero address passed to the constructor there is no way to change it and the pool needs to be reinitialized.

## Proof of Concept
https://github.com/code-423n4/2021-10-tempus/blob/main/contracts/TempusPool.sol#L66-L100

## Recommended Mitigation Steps
Add something along the lines of `require(ctrl != address(0), "controller can not be zero` to avoid potential invalid pool initializations. 

