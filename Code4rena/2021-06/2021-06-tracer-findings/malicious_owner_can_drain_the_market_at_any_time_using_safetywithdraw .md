## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [Malicious owner can drain the market at any time using SafetyWithdraw ](https://github.com/code-423n4/2021-06-tracer-findings/issues/81) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The withdrawERC20Token() in SafetyWithdraw inherited in TracerPerpetualSwaps is presumably a guarded launch emergency withdrawal mechanism. However, given the trust model where the market creator/owner is potentially untrusted/malicious, this is a dangerous approach to emergency withdrawal in the context of guarded launch. 

Alternatively, if this is meant for the owner to withdraw “external” ERC20 tokens mistakenly deposited to the Tracer market then the function should exclude tracerQuoteToken from being the tokenAddress that can be used as a parameter to withdrawERC20Token().

Impact: Malicious owner of a market withdraws/rugs all tracerQuoteTokens deposited at any time after market launch. All users lose deposits. Protocol takes a reputational hit and has to refund the users from treasury.

## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/lib/SafetyWithdraw.sol#L8-L14

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L20


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

For a guarded launch circuit breaker, design a pause/unpause feature where deposits are paused  (in emergency situations) but withdrawals are allowed by the depositors themselves instead of the owner. Alternatively, if this is meant to be for removing external ERC20 tokens accidentally deposited to market, exclude the tracerQuoteToken from being given as the tokenAddress. 

