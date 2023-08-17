## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Twav._getTwav() will return a wrong result when twavObservations[TWAV_BLOCK_NUMBERS - 1].timestamp = 0.](https://github.com/code-423n4/2022-06-nibbl-findings/issues/112) 

# Lines of code

https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/Twav/Twav.sol#L36


# Vulnerability details

## Impact
The "if" condition of Twav._getTwav() is missing some edge cases.
In this case, this function will return 0 which is different from the correct value and it will affect the main functions like NibblVault.buy() and NibblVault.sell().

## Proof of Concept
I think this condition is to confirm at least 4 values were saved for twav calculation.
Btw this timestamp would be zero even though there are more than 4 values properly as it's modularized by 2**32.
In this case, the if condition will be false and this function will return 0.

## Tools Used
Solidity Visual Developer of VSCode

## Recommended Mitigation Steps
I see "cumulativeValuation" is increasing all the time and recommend replacing "timestamp" with "cumulativeValuation".
```
if (twavObservations[TWAV_BLOCK_NUMBERS - 1].cumulativeValuation != 0) {
```

