## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Withdraw from `AaveVault` will receive less than actual share](https://github.com/code-423n4/2021-12-mellow-findings/issues/82) 

# Handle

gzeon


# Vulnerability details

## Impact
`AaveVault` cache `tvl` and update it at the end of each `_push` and `_pull`. When withdrawing from `LpIssuer`,  `tokenAmounts` is calculated using the cached `tvl` to be pulled from `AaveVault`. This will lead to user missing out their share of the accrued interest / donations to Aave since the last `updateTvls`.

## Proof of Concept
https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/LpIssuer.sol#L150
https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/AaveVault.sol#L13

## Recommended Mitigation Steps
Call `updateTvls` at the beginning of `withdraw` function if the `_subvault` will cache tvl

