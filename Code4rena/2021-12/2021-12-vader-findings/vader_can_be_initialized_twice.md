## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- LiquidityBasedTWAP

# [vader can be initialized twice](https://github.com/code-423n4/2021-12-vader-findings/issues/139) 

# Handle

danb


# Vulnerability details

https://github.com/code-423n4/2021-12-vader/blob/main/contracts/lbt/LiquidityBasedTWAP.sol#L221
vader can be initialized twice if in the first call to `setupVader`, `vaderPrice == 0`.

## Recommended Mitigation Steps
add:
```
require(vaderPrice > 0);
```
in `setupVader`.

