## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Inheritance from BaseGeneralPool is unused](https://github.com/code-423n4/2021-10-tempus-findings/issues/17) 

# Handle

TomFrench


# Vulnerability details

## Impact

## Proof of Concept

As the `TempusAmm` only ever registers with the Balancer Vault with the two token specialization the `GeneralPool` interface will never be used as the Vault will call the `MinimalSwapInfoPool` hooks instead.

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/amm/TempusAMM.sol#L107-L110

See here in the Balancer Vault code:

https://github.com/balancer-labs/balancer-v2-monorepo/blob/62c5cba7cae1d481f913c90fe0d9d94e101570c5/pkg/vault/contracts/Swaps.sol#L287-L292

## Recommended Mitigation Steps

Remove the inheritance from `BaseGeneralPool` and remove the functions highlighted in the link below. This will help reduce bytecode from the AMM factory and reduce deployment costs.

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/amm/TempusAMM.sol#L283-L309

