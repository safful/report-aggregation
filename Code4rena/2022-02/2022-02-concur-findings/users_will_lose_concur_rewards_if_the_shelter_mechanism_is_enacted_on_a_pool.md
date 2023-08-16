## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Users Will Lose Concur Rewards If The Shelter Mechanism Is Enacted On A Pool](https://github.com/code-423n4/2022-02-concur-findings/issues/116) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol
https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/MasterChef.sol


# Vulnerability details

## Impact

The shelter mechanism aims to protect the protocol's users by draining funds into a separate contract in the event of an emergency. However, while users are able to reclaim their funds through the `Shelter.sol` contract, they will still have a deposited balance from the perspective of `ConvexStakingWrapper.sol`.

However, if the shelter mechanism is enacted before users are able to claim their Concur rewards, any accrued tokens will be lost and the `MasterChef.sol` contract will continue to allocate tokens to the sheltered pool which will be forever locked within this contract.

There is currently no way to remove sheltered pools from the `MasterChef.sol` contract, hence any balance lost in the contract cannot be recovered due to a lack of a sweep mechanism which can be called by the contract owner.

## Proof of Concept

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/MasterChef.sol

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider removing sheltered pools from the `MasterChef.sol` Concur token distribution. It is important to ensure `massUpdatePools` is called before making any changes to the list of pools. Additionally, removing pools from this list may also create issues with how `_pid` is produced on each new pool. Therefore, it may be worthwhile to rethink this mechanism such that `_pid` tracks some counter variable and not `poolInfo.length - 1`.

