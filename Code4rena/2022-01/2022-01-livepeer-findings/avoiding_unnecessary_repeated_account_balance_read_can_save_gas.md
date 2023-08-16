## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Avoiding unnecessary repeated account balance read can save gas](https://github.com/code-423n4/2022-01-livepeer-findings/issues/135) 

# Handle

WatchPug


# Vulnerability details

https://github.com/livepeer/protocol/blob/20e7ebb86cdb4fe9285bf5fea02eb603e5d48805/contracts/token/BridgeMinter.sol#L90-L98

```solidity
function withdrawETHToL1Migrator() external onlyL1Migrator returns (uint256) {
    uint256 balance = address(this).balance;

    // call() should be safe from re-entrancy here because the L1Migrator and l1MigratorAddr are trusted
    (bool ok, ) = l1MigratorAddr.call.value(address(this).balance)("");
    require(ok, "BridgeMinter#withdrawETHToL1Migrator: FAIL_CALL");

    return balance;
}
```

At L94, `address(this).balance` can be replaced with `balance` to avoid unnecessarily repeated read of account balance state to save some gas.


