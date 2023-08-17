## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Users can claim extremely large rewards or lock rewards from LpGauge due to uninitialised `poolLastUpdate` variable](https://github.com/code-423n4/2022-05-backd-findings/issues/141) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/LpGauge.sol#L115-L119
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/StakerVault.sol#L326-L328


# Vulnerability details

## Impact
A user can claim all of the available governance tokens or prevent any rewards from being claimed in `LpGauge.sol` if sufficient time is left between deploying the contract and initialising it in the `StakerVault.sol` contract by calling `initalizeLPGauge()` OR if a new `LPGauge` contract is deployed and added to `StakerVault` using `prepareLPGauge`.

Inside `LPGauge.sol` when calling `_poolCheckPoint()`, the `lastUpdated` variable is not initalised so defaults to a value of `0`, therefore if the user has managed to stake tokens in the `StakerVault` then the calculated `poolStakedIntegral` will be very large (as block.timestamp is very large). Therefore a user can mint most current available governance tokens for themselves when they claim their rewards (or prevent any governance tokens from being claimed).

## Proof of Concept
1. LP Gauge and StakerVault contracts are deployed
2. Before the `initializeLpGauge()`, user A will stake 1 token with `stakeFor()` thereby increasing `_poolTotalStaked` by 1.
As the `lpgauge` address is equal to the zero address, `_userCheckPoint()` will not be called and `poolLastUpdate` will remain at 0.
3. The user can then directly call `_userCheckPoint()` and be allocated a very large number of shares. This works because `poolLastUpdate` is 0 but the staked amount in the vault is larger than 0
4. Once `initializeLPGauge()` is called, the user can then call `claimRewards()` and receive a very large portion of tokens or if `poolStakedIntegral` exceeds the mint limit set by `Minter.sol` then no one else can claim governance tokens from the lpGauge.

OR
5. A new LP Gauge contract is deployed and added to the vault using `prepareGauge()`.
Follow steps 2 to 4.

## Tools Used
Manual audit
## Recommended Mitigation Steps
Initialise `poolLastUpdate` in the constructor

