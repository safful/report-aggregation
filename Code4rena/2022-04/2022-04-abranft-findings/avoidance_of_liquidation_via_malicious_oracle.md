## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Avoidance of Liquidation Via Malicious Oracle](https://github.com/code-423n4/2022-04-abranft-findings/issues/136) 

# Lines of code

https://github.com/code-423n4/2022-04-abranft/blob/5cd4edc3298c05748e952f8a8c93e42f930a78c2/contracts/NFTPairWithOracle.sol#L312-L318


# Vulnerability details

Issue: Arbitrary oracles are permitted on construction of loans, and there is no check that the lender agrees to the used oracle.

Consequences: A borrower who requests a loan with a malicious oracle can avoid legitimate liquidation.

## Proof of Concept

- Borrower requests loan with an malicious oracle
- Lender accepts loan unknowingly
- Borrowers's bad oracle is set to never return a liquidating rate on `oracle.get` call.
- Lender cannot call `removeCollateral` to liquidate the NFT when it should be allowed, as it will fail the check on [L288](https://github.com/code-423n4/2022-04-abranft/blob/5cd4edc3298c05748e952f8a8c93e42f930a78c2/contracts/NFTPairWithOracle.sol#L288)
- To liquidate the NFT, the lender would have to whitehat along the lines of H-01, by atomically updating to an honest oracle and calling `removeCollateral`.

## Mitigations

- Add `require(params.oracle == accepted.oracle)` as a condition in `_lend`
- Consider only allowing whitelisted oracles, to avoid injection of malicious oracles at the initial loan request stage


