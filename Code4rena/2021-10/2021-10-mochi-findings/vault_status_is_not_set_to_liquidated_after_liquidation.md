## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Vault status is not set to Liquidated after liquidation](https://github.com/code-423n4/2021-10-mochi-findings/issues/51) 

# Handle

gzeon


# Vulnerability details

## Impact
There is a status enum Liquidated but was not used anywhere in the code. 

## Proof of Concept
https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/vault/MochiVault.sol
L277-296 status was not set to Status.Liquidated after liquidation

## Tools Used
None

## Recommended Mitigation Steps
details[id].status = Status.Liquidated;

