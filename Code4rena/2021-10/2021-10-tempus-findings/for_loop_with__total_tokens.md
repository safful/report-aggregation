## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [for loop with _TOTAL_TOKENS](https://github.com/code-423n4/2021-10-tempus-findings/issues/36) 

# Handle

pauliax


# Vulnerability details

## Impact
The loop here is not really necessary as _TOTAL_TOKENS is a constant of 2 so there is always just 1 iteration:
        for (uint256 i = 1; i < _TOTAL_TOKENS; ++i) {
            uint256 currentBalance = balances[i];
            if (currentBalance > maxBalance) {
                chosenTokenIndex = i;
                maxBalance = currentBalance;
            }
        }

## Recommended Mitigation Steps
Consider if you want to reduce gas usage by eliminating this loop here but taking the risk that _TOTAL_TOKENS will not be updated to a different value.

