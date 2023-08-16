## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Inconsistent value of burnSynth between Pool and Synth](https://github.com/code-423n4/2021-07-spartan-findings/issues/70) 

# Handle

jonah1005


# Vulnerability details

## Impact
When users try to born synth, the fee and the value of Sparta is calculated at contract `Pool` while the logic of burning `Pool`s Lp and Synth is located at `Synth` contract.

Users can send synth to the `Synth` contract directly and trigger `burnSynth` at the `Pool` contract. The Pool would not send any token out while the `Synth` contract would burn the lp and Synth.
While users can not drain the liquidity by doing this, breaking the AMM rate unexpectedly is may lead to troubles.  The calculation of debt and the fee would end up with a wrong answer.

## Proof of Concept
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L245

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Synth.sol#L174

## Tools Used
None

## Recommended Mitigation Steps

Pool's `burnSynth` and Synth's `burnSynth` are tightly coupled functions. In fact, according to the current logic, `Synth:burnSynth` should only be triggered from a valid `Pool` contract.

IMHO, applying the`Money in - Money Out` model in the `Synth` contract does more harm than good to the readability and security of the protocol. Consider to let `Pool` contract pass the parameters to the `Synth` contract and add a require check in the `Synth` contract.



