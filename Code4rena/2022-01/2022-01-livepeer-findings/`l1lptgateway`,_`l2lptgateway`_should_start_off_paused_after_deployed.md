## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [`L1LPTGateway`, `L2LPTGateway` should start off paused after deployed](https://github.com/code-423n4/2022-01-livepeer-findings/issues/203) 

# Handle

WatchPug


# Vulnerability details

Per the document: https://github.com/livepeer/LIPs/blob/master/LIPs/LIP-73.md#upgrade-process

> *Phase 1*
>
> - The L1 RoundsManager will be upgraded to disable round initialization at `LIP_73_ROUND`
> - During this phase, protocol transactions will be executed normally
> - During this phase, the following contracts will be deployed:
>     - Protocol contracts on L2
>     - Migrator contracts on L1 and L2
>     - LPT bridge contracts on L1 and L2
>     - ***All of these contracts will start off paused***

However, the current implementation of `L1LPTGateway`, `L2LPTGateway` are not automatically paused on deployment.

We recommend adding `_pause()` to the end of the `constructor()` in `L1LPTGateway`, `L2LPTGateway`, like the constructor of [L1Migrator.sol#L143](https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L143-L143), and `unpause()` when Phase 2 starts. 

This will help avoid tx to happen in an intermediate state between Phase1 and Phase 2, which may cause certain txs to fail, for instance:

When in Phase 1, `L1LPTGateway` cant calls `bridgeMint()` on the `BridgeMinter` to mint LPT to the user, as L1 Minter have not `migrateToNewMinter()` to `BridgeMinter` yet. If a user in L2 tries to move `LPT` from L2 to L1, their tx may fail.

