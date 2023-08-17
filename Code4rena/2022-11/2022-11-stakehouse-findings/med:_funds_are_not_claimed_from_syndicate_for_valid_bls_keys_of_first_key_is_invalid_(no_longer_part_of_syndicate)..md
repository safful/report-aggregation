## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-28

# [MED: Funds are not claimed from syndicate for valid BLS keys of first key is invalid (no longer part of syndicate).](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/408) 

# Lines of code

 https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L218


# Vulnerability details

## Description

claimRewards in StakingFundsVault.sol has this code:
```
if (i == 0 && !Syndicate(payable(liquidStakingNetworkManager.syndicate())).isNoLongerPartOfSyndicate(_blsPubKeys[i])) {
    // Withdraw any ETH accrued on free floating SLOT from syndicate to this contract
    // If a partial list of BLS keys that have free floating staked are supplied, then partial funds accrued will be fetched
    _claimFundsFromSyndicateForDistribution(
        liquidStakingNetworkManager.syndicate(),
        _blsPubKeys
    );
    // Distribute ETH per LP
    updateAccumulatedETHPerLP();
}
```

The issue is that if the first BLS public key is not part of the syndicate, then \_claimFundsFromSyndicateForDistribution will not be called, even on BLS keys that are eligible for syndicate rewards. This leads to reduced rewards for user.

This is different from a second bug which discusses the possibility of using a stale acculmulatedETHPerLP.

## Impact

Users will not receive rewards for claims of valid public keys if first passed key is not part of syndicate.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Drop the i==0 requirement, which was intended to make sure the claim isn't called multiple times. Use a hasClaimed boolean instead.