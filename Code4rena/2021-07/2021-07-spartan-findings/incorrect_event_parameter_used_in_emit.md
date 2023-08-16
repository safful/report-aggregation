## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Incorrect event parameter used in emit](https://github.com/code-423n4/2021-07-spartan-findings/issues/119) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Incorrect event parameter outputAmount is used (instead of output) in the MintSynth event emit. outputAmount is a named return variable that is never set in this function and so will always be 0. This should instead be output. This will confuse the UI or offchain monitoring tools that 0 synths were minted and will lead to users panicking/complaining or trying to mint synth again.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L240

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L229

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L232

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Replace outputAmount with output in the emit.

