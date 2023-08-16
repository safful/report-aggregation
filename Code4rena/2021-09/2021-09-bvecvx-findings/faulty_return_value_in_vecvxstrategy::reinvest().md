## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Faulty return value in veCVXStrategy::reinvest()](https://github.com/code-423n4/2021-09-bvecvx-findings/issues/12) 

# Handle

pmerkleplant


# Vulnerability details

## Impact
The function `reinvest` in the veCVXStrategy always returns 0 as the return variable `reinvested` is never updated. 
The function is `onlyGovernance` and the return value probably does not matter if the caller is a multi-sig. However, if a protocol is set as `onlyGovernance` the faulty return value would have to be ignored by the caller to not transition into an incorrect state.

## Proof of Concept
The variable `reinvested` is declared as return variable (line 400) but not updated to reflect the actual amount reinvested which is saved in variable `toDeposit`.

Therefore always the default value is returned (0).

Link: https://github.com/code-423n4/2021-09-bvecvx/blob/32ecfd005d421f29c3846f4609fec33eaad388b9/veCVX/contracts/veCVXStrategy.sol#L400

## Recommended Mitigation Steps

Add `reinvested = toDeposit;` after line 412.

