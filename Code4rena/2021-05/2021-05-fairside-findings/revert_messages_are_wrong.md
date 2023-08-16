## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Revert messages are wrong](https://github.com/code-423n4/2021-05-fairside-findings/issues/64) 

# Handle

s1m0


# Vulnerability details

## Impact
The following revert messages refer to a different function instead of the one where they actually are, making harder to understand the flow of the program in case of error.
[l. 166](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/token/FSD.sol#L166)
[l. 185](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/token/FSD.sol#L185)
[l. 254](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/token/FSD.sol#L254)

## Recommended Mitigation Steps
Set the messages with the correct function name.

