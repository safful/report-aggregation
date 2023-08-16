## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Renaming variables for clarity](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/125) 

# Handle

nathaniel


# Vulnerability details

## Impact
`amount` in the `Investors` struct is vague. It would be assumed to be the invested amount, however this amount decreases when the beneficiary claims. A more appropriate name could be `unclaimed_amount`.
`claimable_to_send` is not appropriate name in the `claimExact()` function, as it is not the claimable total, instead `exact_claim_to_send` would make more sense.
## Proof of Concept
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L24
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L155


