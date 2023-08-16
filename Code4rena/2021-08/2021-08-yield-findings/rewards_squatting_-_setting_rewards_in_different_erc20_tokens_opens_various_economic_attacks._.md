## Tags

- bug
- duplicate
- 2 (Med Risk)
- sponsor confirmed
- ERC20Rewards

# [Rewards squatting - setting rewards in different ERC20 tokens opens various economic attacks. ](https://github.com/code-423n4/2021-08-yield-findings/issues/64) 

# Handle

moose-code


# Vulnerability details

## Impact
Users have essentially have an option to either claim currently earned reward amounts on future rewards tokens, or the current rewards token. 

Although stated on line 84, it does not take into account the implications and lock in this contract will have on the future value of new tokens able to be issued via rewards. 

## Proof of Concept
Smart users will monitor the mempool for setRewards transactions. If the new reward token (token b) is less valuable than the old reward token (token a), they front run this transaction by calling claim. Otherwise they let their accrued 'token a' roll into rewards of of the more valuable 'token b'.

Given loads of users will likely hold these tokens from day 1, there will potentially be thousands of different addresses squatting on rewards. 

Economically, given the above it makes sense that the value of new reward tokens, i.e. 'token b' should always be less than that of 'token a'. This is undesirable in a rewards token contract as there is no reliable way to start issuing a more valuable token at a later stage, unless exposing yourself to a major risk of reward squatting.

i.e. You could not in future say we want to run a rewards period of of issuing an asset like WETH rewards for 10 days, after first initially issuing DAI as a reward. This hamstrings flexibility of the contract.

P.s. This is one of the slickest contracts I've read. Love how awesome it is.Just believe this should be fixed, then its good to go. 

## Tools Used
Manual analysis

## Recommended Mitigation Steps
It is true you could probably write a script to manually go call 'claim' on thousands of squatting token addresses but this is a poor solution. 

A simple mapping pattern could be used with an index mapping to a reward cycle with a reward token and a new accumulative etc. Users would likely need to be given a period a to claim from old reward cycles before their token balance could no longer reliably used to calculate past rewards. The would still be able to claim everything up until their last action (even though this may be before the rewards cycle ended).

