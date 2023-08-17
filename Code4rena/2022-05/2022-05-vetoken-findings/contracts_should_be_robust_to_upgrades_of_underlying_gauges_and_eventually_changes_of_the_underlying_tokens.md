## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Contracts should be robust to upgrades of underlying gauges and eventually changes of the underlying tokens](https://github.com/code-423n4/2022-05-vetoken-findings/issues/50) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/VoterProxy.sol


# Vulnerability details

## Impact

For some veAsset project (for example Angle’s [gauges](https://github.com/AngleProtocol/angle-core/blob/main/contracts/staking/LiquidityGaugeV4UpgradedToken.vy), gauge contracts are upgradable, so interfaces and underlying LP tokens are subject to change, blocking and freezing the system. Note that this is not hypothetic as it happened a few weeks ago: see this [snapshot vote](
https://snapshot.org/#/anglegovernance.eth/proposal/0x1adb0a958220b3dcb54d2cb426ca19110486a598a41a75b3b37c51bfbd299513). Therefore, the system should be robust to a change in the pair gauge / token. 

Note that is doable in the current setup for the veToken team to rescue the funds in such case, hence it is only a medium issue.
You’d have to do as follow: a painful shutdown of the `Booster` (which would lead to an horrible situation where you’d have to preserve backwards compatibility for LPs to save their funds in the new Booster), an operator change in `VoterProxy` to be able to call `execute`.

## Mitigation steps
To deal with upgradeable contracts, either the `VoterProxy` needs to be upgradable to deal with any situation that may arise, either you need to add upgradeable “intermediate” contracts between the `staker` and the gauge that could be changed to preserve the logic.

