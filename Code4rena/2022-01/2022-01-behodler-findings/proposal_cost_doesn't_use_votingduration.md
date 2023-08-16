## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Proposal cost doesn't use votingDuration](https://github.com/code-423n4/2022-01-behodler-findings/issues/189) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The LimboDAO.sol contract allows the votingDuration to be modified in the [`setProposalConfig()` function](https://github.com/code-423n4/2022-01-behodler/blob/cedb81273f6daf2ee39ec765eef5ba74f21b2c6e/contracts/DAO/LimboDAO.sol#L302), but the `makeProposal()` function hard codes this value as two days, which is the initialized value, in the `makeProposal()` fee calculation.

## Proof of Concept

First, observe the comment on line 209 has a comment:
```
proposalConfig.requiredFateStake = 223 * ONE; //50000 EYE for 24 hours
```

This comment indicates that the quantity of 50,000 EYE is needed for each day. The votingDuration value [is initialized to 2 days](https://github.com/code-423n4/2022-01-behodler/blob/cedb81273f6daf2ee39ec765eef5ba74f21b2c6e/contracts/DAO/LimboDAO.sol#L208). Later in the code, the "proposalConfig.requiredFateStake" variable is multiplied by 2. Although there is no explanation for this value, given the earlier comment that the "proposalConfig.requiredFateStake" value is required every day, the cost to make a proposal should vary based on the current votingDuration value:

```
fateState[proposer].fateBalance = fateState[proposer].fateBalance - proposalConfig.requiredFateStake * 2;
```

Because a constant value of 2 is used, most likely assuming a constant 2 day votingDuration, later modifications to the votingDuration will not change the cost of making a proposal. If the votingDuration increases, the proposal cost will be less EYE per hour, and if the votingDuration decreases, the proposal cost will be more EYE per hour.

## Recommended Mitigation Steps

One of two portions of the code is wrong and needs modification:
1. The comment about "50000 EYE for 24 hours" is wrong because it doesn't take into account the variability of votingDuration. Even if the comment only refers to the initialized values, it should state "50000 EYE for 48 hours" because the votingDuration is 2 days.
2. The `makeProposal()` function calculates the fate cost incorrectly because it never uses the votingDuration variables in its calculation.

