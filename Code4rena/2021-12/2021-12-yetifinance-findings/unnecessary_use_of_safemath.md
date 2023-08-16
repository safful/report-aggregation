## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary use of Safemath](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/85) 

# Handle

Jujic


# Vulnerability details

## Impact
```
deploymentTime = block.timestamp;
uint public constant BOOTSTRAP_PERIOD = 14 days;

deploymentTime.add(BOOTSTRAP_PERIOD) // doesn't overflow

```

## Proof of Concept
https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/BorrowerOperations.sol#L899

## Tools Used
Remix
## Recommended Mitigation Steps
I recommend   not  use Safemath for this operation.

