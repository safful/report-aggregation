## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Lack of require() allows control flow to proceed leading to undefined behavior](https://github.com/code-423n4/2021-07-spartan-findings/issues/131) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The _handleTransferIn() functions use a conditional check (_amount > 0) to execute the transfer-in logic of tokens. This should really be a require() to prevent zero amount transfers into the protocol which will allow subsequent logic to execute and potentially utilize any dust/stuck funds from earlier to be accounted to the sender.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L198-L210

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Synth.sol#L202-L206

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/poolFactory.sol#L110-L114

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change condition check to a require() which will revert any transfers of zero tokens/funds.

