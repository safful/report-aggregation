## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [setReward does not check if pid exists](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/70) 

# Handle

pauliax


# Vulnerability details

## Impact
function setReward does not check if pid actually exists. Provided wrong _pair, _token and _isSupply params it will return a default value of 0, thus the first pool will be updated even though the caller may intended to update another pool. The risk is very low as this function can only be called by onlyOwner but I still think the code should prevent such scenarios from accidentally happening.

## Recommended Mitigation Steps
Check that pidByPairToken added value is true.

