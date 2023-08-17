## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Might not get desired min loan amount if `_originationFeeRate` changes](https://github.com/code-423n4/2022-04-backed-findings/issues/28) 

# Lines of code

https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L309


# Vulnerability details

## Impact
Admins can update the origination fee by calling `updateOriginationFeeRate`.
Note that a borrower does not receive their `minLoanAmount` set in `createLoan`, they only receive `(1 - originationFee) * minLoanAmount`, see [`lend`](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L159).
Therefore, they need to precalculate the `minLoanAmount` using the **current** origination fee to arrive at the post-fee amount that they actually receive.
If admins then increase the fee, the borrower receives fewer funds than required to cover their rent and might become homeless.

## Recommended Mitigation Steps
Reconsider how the min loan amount works. Imo, this `minLoanAmount` should be the post-fee amount, not the pre-fee amount. It's also more intuitive for the borrower when creating the loan.


