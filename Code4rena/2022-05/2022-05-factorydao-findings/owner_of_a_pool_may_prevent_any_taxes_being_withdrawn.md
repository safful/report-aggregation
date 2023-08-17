## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Owner of a pool may prevent any taxes being withdrawn](https://github.com/code-423n4/2022-05-factorydao-findings/issues/52) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L261-L272


# Vulnerability details

## Impact

It is possible for the owner of a pool to prevent any taxes being withdrawn by the `globalBeneficiary`. The impact is the taxed tokens will be permanently locked in the contract and `withdrawTaxes()` will not be callable for that `poolId`.


## Proof of Concept

The attack works by setting one of the `rewardTokenAddresses` to a malicious contract during `addPool()`. The malicious contract is set such that it will revert on the call `pool.rewardTokens[i]).transfer(globalBeneficiary, tax)` if an only if the `to` address is `globalBeneficiary.

The result of this attack is that if one reward transfer fails then entire `withdrawTaxes()` transaction will revert and no taxes can be claimed. However, the pool will function correctly for all other users.

```solidity
    function withdrawTaxes(uint poolId) external {
        Pool storage pool = pools[poolId];
        require(pool.id == poolId, 'Uninitialized pool');


        bool success = true;
        for (uint i = 0; i < pool.rewardTokens.length; i++) {
            uint tax = taxes[poolId][i];
            taxes[poolId][i] = 0;
            success = success && IERC20(pool.rewardTokens[i]).transfer(globalBeneficiary, tax);
        }
        require(success, 'Token transfer failed');
    }
```

## Recommended Mitigation Steps

There are a few mitigations to this issue.

The first is for the `withdrawTaxes()` function to take both `poolId` and `rewardIndex` as a parameters to allowing the tax beneficiary to only withdraw from certain reward tokens in the pool. This would allow the beneficiary to withdraw from all reward tokens except malicious ones.

The second mitigation is to implement a `try-catch` condition around the withdrawal of reward tokens. In the catch statement re-instate the `taxes[poolId][i] = tax` if the transfer fails. Alternatively just skip the reward tokens if the transfer fails though this would be undesirable if a token is paused for some reason.

