## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- resolved
- sponsor confirmed

# [Incorrect require statement](https://github.com/code-423n4/2022-01-behodler-findings/issues/213) 

# Handle

csanuragjain


# Vulnerability details

## Impact


## Proof of Concept

1. Navigate to contract at https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/UniswapHelper.sol

2. Observe the configure function which has below require condition

```
require(priceBoostOvershoot < 100, "Set overshoot to number between 1 and 100.");
```

3. This means priceBoostOvershoot can be set to 0 which contradicts the require statement message mentioning "Set overshoot to number between 1 and 100."
## Tools Used

## Recommended Mitigation Steps
Change the require condition to 

```
require(priceBoostOvershoot < 100 && priceBoostOvershoot > 0, "Set overshoot to number between 1 and 100.");
```

