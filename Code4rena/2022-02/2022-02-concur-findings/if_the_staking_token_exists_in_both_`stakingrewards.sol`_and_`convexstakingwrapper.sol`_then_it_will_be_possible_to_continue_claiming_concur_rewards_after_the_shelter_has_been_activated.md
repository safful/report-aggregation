## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [If The Staking Token Exists In Both `StakingRewards.sol` And `ConvexStakingWrapper.sol` Then It Will Be Possible To Continue Claiming Concur Rewards After The Shelter Has Been Activated](https://github.com/code-423n4/2022-02-concur-findings/issues/117) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol
https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/MasterChef.sol
https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/StakingRewards.sol


# Vulnerability details

## Impact

Staking tokens are used to deposit into the `StakingRewards.sol` and `ConvexStakingWrapper.sol` contracts. Once deposited, the user is entitled to Concur rewards in proportion to their staked balance and the underlying pool's `allocPoint` in the `MasterChef.sol` contract.

The `Shelter.sol` mechanism allows the owner of the `ConvexStakingWrapper.sol` to react to emergency events and protect depositor's assets. The staking tokens can be withdrawn after the grace period has passed. However, these staking tokens can be deposited into the `StakingRewards.sol` contract to continue receiving Concur rewards not only for `StakingRewards.sol` but also for their `ConvexStakingWrapper.sol` deposited balance which has not been wiped. As a result, users are able to effectively claim double the amount of Concur rewards they should be receiving.

## Proof of Concept

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/MasterChef.sol

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/StakingRewards.sol

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Ensure that staking tokens cannot be deposited in both the `StakingRewards.sol` and `ConvexStakingWrapper.sol` contracts. If this is intended behaviour, it may be worthwhile to ensure that the sheltered users have their deposited balance wiped from the `MasterChef.sol` contract upon being sheltered.

