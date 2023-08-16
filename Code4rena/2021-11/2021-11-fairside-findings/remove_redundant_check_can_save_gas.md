## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Remove redundant check can save gas](https://github.com/code-423n4/2021-11-fairside-findings/issues/56) 

# Handle

WatchPug


# Vulnerability details

The check if `_wallets.length <= 2` is redundant as the length of `_wallets` parameter must be 2.

https://github.com/code-423n4/2021-11-fairside/blob/20c68793f48ee2678508b9d3a1bae917c007b712/contracts/network/FSDNetwork.sol#L520-L533

```solidity=520
function setMembershipWallets(address[2] calldata _wallets) external {
    //todo internal
    require(
        membership[msg.sender].wallets[0] == address(0) &&
            membership[msg.sender].wallets[1] == address(0),
        "FSDNetwork::setMembershipWallets: Cannot have more than three wallets per membership"
    );
    require(
        _wallets.length <= 2,
        "FSDNetwork::setMembershipWallets: Too many wallets"
    );
    membership[msg.sender].wallets = _wallets;
}
```

### Recommendation

Remove the redundant check.

