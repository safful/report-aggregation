## Tags

- bug
- question
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [You can flip governance decisions without extending vote duration](https://github.com/code-423n4/2022-01-behodler-findings/issues/106) 

# Handle

camden


# Vulnerability details

## Impact
The impact here is that a user can, right at the end of the voting period, flip the decision without triggering the logic to extend the vote duration. The user doesn't even have to be very sophisticated: they can just send one vote in one transaction to go to 0, then in a subsequent transaction send enough to flip the vote.

## Proof of Concept
https://github.com/code-423n4/2022-01-behodler/blob/608cec2e297867e4d954a63fecd720e80c1d5ae8/contracts/DAO/LimboDAO.sol#L281
You can send exactly enough fate to send the fate amount to 0, then send fate to change the vote. You'll never trigger this logic.

On the first call, to send the currentProposalState.fate to 0, `(fate + currentFate) * fate == 0`, so we won't extend the proposal state.

Then, on the second call, to actually change the vote, `fate * currentFate == 0` because `currentFate` is 0. 

## Recommended Mitigation Steps
Make sure that going to 0 is equivalent to a flip, but going away from 0 isn't a flip.

