## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [The    initialize    function   does   not   check   for   non-zero   address and emit event](https://github.com/code-423n4/2022-01-livepeer-findings/issues/200) 

# Handle

Jujic


# Vulnerability details

## Impact
The    initialize    function   does   not   check   if   the   `_bondingManager`   are   all   non-zero   addresses.   If   all  the   initialized   `_bondingManager`   happen   to   be   0,   the   contract   will   have   to   be redeployed.

The  contract are initialized, but their critical init parameters are not logged for any off-chain monitoring.

## Proof of Concept
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/pool/DelegatorPool.sol#L47-L51

```
function initialize(address _bondingManager) public initializer {
        bondingManager = _bondingManager;
        migrator = msg.sender;
        initialStake = pendingStake();
    }
```
Most contracts use initialize() functions instead of constructor given the delegatecall proxy pattern. While most of them emit an event in the critical initialize() functions to record the init parameters for off-chain monitoring and transparency reasons, DelegatorPool.sol not emit such an event in their initialize() function.



## Tools Used
https://github.com/code-423n4/2021-06-pooltogether-findings/issues/68
## Recommended Mitigation Steps
Add check for zero address and emit event.

