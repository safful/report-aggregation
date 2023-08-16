## Tags

- bug
- 2 (Med Risk)
- sponsor acknowledged
- sponsor confirmed

# [Rewards can be stolen](https://github.com/code-423n4/2021-12-nftx-findings/issues/136) 

# Handle

cmichel


# Vulnerability details

The `NFTXInventoryStaking` contract distributes new rewards to all previous stakers when the owner calls the `receiveRewards` function.
This allows an attacker to frontrun this `receiveRewards` transaction when they see it in the mem pool with a `deposit` function.
The attacker will receive the rewards pro-rata to their deposits.
The deposit will be locked for 2 seconds only (`DEFAULT_LOCKTIME`) after which the depositor can withdraw their initial deposit & the rewards again for a profit.

The rewards can be gamed this way and one does not actually have to _stake_, only be in the staking contract at the time of reward distribution for 2 seconds.
The rest of the time they can be used for other purposes.

## Recommended Mitigation Steps
Distribute the rewards equally over time to the stakers instead of in a single chunk on each `receiveRewards` call.
This is more of a "streaming rewards" approach.


