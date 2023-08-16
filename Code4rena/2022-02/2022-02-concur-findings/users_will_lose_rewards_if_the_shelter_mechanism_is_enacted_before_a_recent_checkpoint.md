## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Users Will Lose Rewards If The Shelter Mechanism Is Enacted Before A Recent Checkpoint](https://github.com/code-423n4/2022-02-concur-findings/issues/115) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol


# Vulnerability details

## Impact

The shelter mechanism aims to protect the protocol's users by draining funds into a separate contract in the event of an emergency. However, while users are able to reclaim their funds through the `Shelter.sol` contract, they will still have a deposited balance from the perspective of `ConvexStakingWrapper.sol`.

Because users will only receive their rewards upon depositing/withdrawing their funds due to how the checkpointing mechanism works, it is likely that by draining funds to the `Shelter.sol` contract, users will lose out on any rewards they had accrued up and until that point. These rewards are unrecoverable and can potentially be locked within the contract if the reward token is unique and only belongs to the sheltered `_pid`.

## Proof of Concept

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider allowing users to call a public facing `_checkpoint` function once their funds have been drained to the `Shelter.sol` contract. This should ensure they receive their fair share of rewards. Careful consideration needs to be made when designing this mechanism, as by giving users full control of the `_checkpoint` function may allow them to continue receiving rewards after they have withdrawn their LP tokens.

