## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Bad redirects can make it impossible to deposit & withdraw](https://github.com/code-423n4/2021-12-mellow-findings/issues/44) 

# Handle

cmichel


# Vulnerability details

The `GatewayVault._push()` function gets `redirects` from the `strategyParams`.
If `redirects[i] = j`, vault index `i`'s deposits are redirected to vault index `j`.

Note that the deposits for vault index `i` are cleared, as they are redirected:

```solidity
for (uint256 j = 0; j < _vaultTokens.length; j++) {
    uint256 vaultIndex = _subvaultNftsIndex[strategyParams.redirects[i]];
    amountsByVault[vaultIndex][j] += amountsByVault[i][j];
    amountsByVault[i][j] = 0;
}
```

> The same is true for withdrawals in the `_pull` function. Users might not be able to withdraw this way.

If the `redirects` array is misconfigured, it's possible that all `amountsByVault` are set to zero.
For example, if `0` redirects to `1` and `1` redirects to `0`. Or `0` redirects to itself, etc.
There are many misconfigurations that can lead to not being able to deposit to the pool anymore.


## Recommended Mitigation Steps
The `redirects[i] = j` matrix needs to be restricted.
If `i` is redirected to `j`, `j` may not redirect itself.
Check for this when setting the `redirects` array.

