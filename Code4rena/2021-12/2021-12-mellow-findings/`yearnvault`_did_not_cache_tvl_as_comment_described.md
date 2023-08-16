## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`YearnVault` did not cache tvl as comment described](https://github.com/code-423n4/2021-12-mellow-findings/issues/84) 

# Handle

gzeon


# Vulnerability details

## Impact
The comment in
https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/YearnVault.sol#L14
> The TVL of the vault is cached and updated after each deposit withdraw.

But it actually does not cache tvl. This behavior is desired or otherwise would have same issue as `AaveVault`.

## Recommended Mitigation Steps
Remove the cache description in comment. 

