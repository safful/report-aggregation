## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [`approveAndCall` approve `max` amount of token](https://github.com/code-423n4/2021-07-spartan-findings/issues/33) 

# Handle

jonah1005


# Vulnerability details

## Impact
`approveAndCall` approve max allowance to the receiver regardless of the given parameter.

This is far away from what the function name implies. Users would lose all the tokens by using this function.

## Proof of Concept
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L118
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Synth.sol#L113
## Tools Used
None
## Recommended Mitigation Steps
Change to `_approve(msg.sender, recipient, amount); `

