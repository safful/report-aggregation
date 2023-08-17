## Tags

- bug
- 2 (Med Risk)
- judge review requested
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-29

# [Medium: User receives less rewards than they are eligible for if first passed BLS key is inactive](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/410) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L224


# Vulnerability details

## Description

StakingFundsVault has the claimRewards() function to allow users to withdraw profits. 
```
function claimRewards(
    address _recipient,
    bytes[] calldata _blsPubKeys
) external nonReentrant {
    for (uint256 i; i < _blsPubKeys.length; ++i) {
        require(
            liquidStakingNetworkManager.isBLSPublicKeyBanned(_blsPubKeys[i]) == false,
            "Unknown BLS public key"
        );
        // Ensure that the BLS key has its derivatives minted
        require(
            getAccountManager().blsPublicKeyToLifecycleStatus(_blsPubKeys[i]) == IDataStructures.LifecycleStatus.TOKENS_MINTED,
            "Derivatives not minted"
        );
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
        // If msg.sender has a balance for the LP token associated with the BLS key, then send them any accrued ETH
        LPToken token = lpTokenForKnot[_blsPubKeys[i]];
        require(address(token) != address(0), "Invalid BLS key");
        require(token.lastInteractedTimestamp(msg.sender) + 30 minutes < block.timestamp, "Last transfer too recent");
        _distributeETHRewardsToUserForToken(msg.sender, address(token), token.balanceOf(msg.sender), _recipient);
    }
}
```

The issue is that updateAccumulatedETHPerLP() is not guaranteed to be called, which means the ETH reward distribution in \_distribute would use stale value, and users will not receive as many rewards as they should.
`updateAccumulatedETHPerLP` is only called if the first BLS public key is part of the syndicate. However, for the other keys it makes no reason not to use the up to date accumulatedETHPerLPShare value.

## Impact

User receives less rewards than they are eligible for if first passed BLS key is inactive.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Call updateAccumulatedETHPerLP() at the start of the function.