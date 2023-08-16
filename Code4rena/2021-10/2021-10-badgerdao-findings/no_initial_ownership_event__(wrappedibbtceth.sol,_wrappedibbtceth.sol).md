## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [No Initial Ownership Event  (WrappedIbbtcEth.sol, WrappedIbbtcEth.sol)](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/22) 

# Handle

ye0lde


# Vulnerability details

For "core", which can be changed by the governance process, an event is emitted when it is changed from 0 to a hopefully valid value in the initialize function.

In the same initialize function the _governance address itself is not verified nor is there an event emitted showing that the governance address has changed from 0 to a different address.

## Proof of Concept

https://github.com/code-423n4/2021-10-badgerdao/blob/9c0ea7b3b02675211446f6c81750c5f3c0a86370/contracts/WrappedIbbtcEth.sol#L37-L46

Similar but with Oracle instead of Core.
https://github.com/code-423n4/2021-10-badgerdao/blob/9c0ea7b3b02675211446f6c81750c5f3c0a86370/contracts/WrappedIbbtc.sol#L38-L45

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Emit an event reporting governance change or if it is not important to report these initialization events remove the emit for the core initialization.

