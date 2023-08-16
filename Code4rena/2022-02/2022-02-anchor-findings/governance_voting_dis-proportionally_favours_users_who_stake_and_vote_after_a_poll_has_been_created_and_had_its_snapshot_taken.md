## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Governance Voting Dis-proportionally Favours Users Who Stake And Vote After A Poll Has Been Created And Had Its Snapshot Taken](https://github.com/code-423n4/2022-02-anchor-findings/issues/64) 

# Lines of code

https://github.com/code-423n4/2022-02-anchor/blob/main/contracts/anchor-token-contracts/contracts/gov/src/contract.rs#L543-L580
https://github.com/code-423n4/2022-02-anchor/blob/main/contracts/anchor-token-contracts/contracts/gov/src/contract.rs#L582-L665
https://github.com/code-423n4/2022-02-anchor/blob/main/contracts/anchor-token-contracts/contracts/gov/src/staking.rs#L15-L57
https://github.com/code-423n4/2022-02-anchor/blob/main/contracts/anchor-token-contracts/contracts/gov/src/contract.rs#L364-L455


# Vulnerability details

## Impact

Polls are created by targeting the `receive_cw20` function which is queried whenever the contract receives tokens. By setting the hook message to `Cw20HookMsg::CreatePoll`, the sender is able to create a poll, assuming the amount sent satisfies the minimum deposit amount for poll creation. Users can also choose to call `ExecuteMsg::SnapshotPoll` or have it handled automatically when a user casts a vote on the newly created poll.

The snapshot simply sets `a_poll.staked_amount`, which represents the total staked amount within the governance contract at a given block. However, during the voting period, other users can stake tokens and effectively have an increasing influence over the outcome of a given poll. There are no check-pointed balances to ensure that a certain user had staked tokens at the time the poll had its snapshot taken.

This can be abused to skew poll results in favour of users who stake their Anchor tokens after a poll has had its snapshot taken.

## Proof of Concept

Let's assume the share to token exchange rate is `1:1` such that if a user deposits 100 Anchor tokens, they receive 100 shares in return.

Consider the following scenario:

- There are a total of 100 Anchor tokens in the Governance contract.
- Alice creates a poll and executes `ExecuteMsg::SnapshotPoll` such that `a_poll.staked_amount == 100`.
- Bob deposits 10 Anchor tokens through the `Cw20HookMsg::StakeVotingTokens` hook message which increases the contract's total balance to 110 and shares to 110 as the exchange rate is `1:1` upon minting and redeeming shares.
- At this point, the target poll has a `a_poll.staked_amount == 100`, even though there are really 110 Anchor tokens staked.
- As a result, if Bob votes on a poll, they have a 10% degree of influence on the outcome of the poll, even though they have less than 10% of the total staked tokens (i.e. 10/110).
- Therefore, poll voters are actually incentivised to stake tokens after a poll has had its snapshot taken in order to maximise their voting power.

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider implementing a check-pointing mechanism such that when a user casts a vote, the user's staked balance is checked at the block height upon which the snapshot was taken instead of checking its most up-to-date staked balance. This check-pointing behaviour is implemented on Ethereum which has a more restrictive block space. The mechanism will simply store the staker's balance on each stake/unstake action. When user's wish to vote, the protocol will check the balance at a specific block (i.e. the snapshotted block). An example implementation can be found [here](https://github.com/compound-finance/compound-protocol/blob/master/contracts/Governance/Comp.sol#L189-L221).

