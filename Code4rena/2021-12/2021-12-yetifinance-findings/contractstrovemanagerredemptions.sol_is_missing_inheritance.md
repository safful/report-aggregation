## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [contracts/TroveManagerRedemptions.sol is missing inheritance](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/188) 

# Handle

heiho1


# Vulnerability details

## Impact

TroveManagerRedemptions does not inherit contracts/Interfaces/ITroveManagerRedemptions.sol but should.  Note that TroveManager.sol does inherit ITroveManager.  Decoupling an interface from its implementation can lead to code drift and incomplete or incorrect interfaces/implementations.

## Proof of Concept

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/TroveManagerRedemptions.sol#L38

## Tools Used

Slither

## Recommended Mitigation Steps

Declare contract as "TroveManagerRedemptions is TroveManagerBase, ITroveManagerRedemptions"

