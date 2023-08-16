## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [burnAddress is not actually meant to burn anything](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/275) 

# Handle

pauliax


# Vulnerability details

## Impact
burnAddress is hardcoded to 0x03Df4ADDfB568b338f6a0266f30458045bbEFbF2. I see this address is a Gnosis safe multisig. So the eth is not actually burned even though I expected the burn by looking at the code. This confusion happens because the codebase was adopted from Vader protocol but with no actual intention of burning.

## Recommended Mitigation Steps
To reduce this confusion and improve the readability of the codebase you should either rename the burn variables and functions or leave it as it is but comment and document the actual mechanics of the sale.


