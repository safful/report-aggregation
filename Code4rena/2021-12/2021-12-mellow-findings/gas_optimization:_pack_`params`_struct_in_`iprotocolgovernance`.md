## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas Optimization: Pack `Params` struct in `IProtocolGovernance`](https://github.com/code-423n4/2021-12-mellow-findings/issues/59) 

# Handle

gzeon


# Vulnerability details

## Impact
Reduce 1 storage slot by reordering from
https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/interfaces/IProtocolGovernance.sol#L13
```
    struct Params {
        bool permissionless;
        uint256 maxTokensPerVault;
        uint256 governanceDelay;
        address protocolTreasury;
    }
```
to
```
    struct Params {
        bool permissionless;
        address protocolTreasury;
        uint256 maxTokensPerVault;
        uint256 governanceDelay;
    }
```


