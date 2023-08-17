## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-11

# [PirexGmx#migrateReward() may cause users to lose Reward.](https://github.com/code-423n4/2022-11-redactedcartel-findings/issues/249) 

# Lines of code

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L940-L949


# Vulnerability details

## Impact

PirexGmx#migrateReward() may cause users to lose Reward. before PirexRewards.sol set new PirexGmx

## Proof of Concept

The current migration process is: call # completemigration ()-> # migrateward ()

After this method, the producer of PirexRewards.sol contract is still the old PirexGmx. 

At this time, if AutoPxGmx#compound () is called by bot:

AutoPxGmx#compound() -> PirexRewards#.claim() -> old_PirexGmx#claimRewards()

Old_PirexGmx#claimRewards () will return zero rewards  and the reward of AutopXGMX will be lost.


old PirexGmx still can execute
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L824-L828

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L940-L949

## Tools Used

## Recommended Mitigation Steps

There are two ways to solve the problem.
1. set the producer of PirexRewards to the new PirexGmx in completeMigration ().
2. In #migrateReward (), set the old PirexGmx's "pirexRewards" to address(0), so that you can't use the old PirexGmx to get rewards

Simply use the second, such as:
```solidity
    function migrateReward() external whenPaused {
        if (msg.sender != migratedTo) revert NotMigratedTo();
        if (gmxRewardRouterV2.pendingReceivers(address(this)) != address(0))
            revert PendingMigration();

        // Transfer out any remaining base reward (ie. WETH) to the new contract
        gmxBaseReward.safeTransfer(
            migratedTo,
            gmxBaseReward.balanceOf(address(this))
        );
+       pirexRewards ==address(0);   //*** set pirexRewards=0,Avoid claimRewards () being called by mistake.***//
    }
```
```
