## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [2-step change of publisher address and licenseFee does not generate warning event ](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/188) 

# Handle

0xRajeev


# Vulnerability details

## Impact
Another big aspect of a 2-step change, such as done with changePublisher() and changeLicenseFee(), is to generate an event when the new address or license fee is registered for change, pending the timelock duration. This is to warn protocol users that a pending change is upcoming (after the timelock) via offchain signalling so they can monitor/notice and decide to engage/exit based on their perception of the impact from the change.

The current implementation only emits an event when the pending change is enforced but not when it is made pending which does not provide one of the biggest benefits of a 2-step change.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L143-L147

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L161-L165

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Add another event when the new publisher or licenseFee is made pending.

