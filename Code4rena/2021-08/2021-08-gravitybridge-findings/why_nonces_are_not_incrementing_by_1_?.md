## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Why nonces are not incrementing by 1 ?](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/32) 

# Handle

pauliax


# Vulnerability details

## Impact
I am concerned why invalidationId, invalidationNonce or valsetNonce are only required to be greater than the previous value. Why did you choose this approach instead of just simply asking for an incremented value? While this may not be a problem if the validators are honest, but otherwise, they may submit a nonce of MAX UINT and thus block the whole system as it would be no longer possible to submit a greater value. Again, just wanted you to be aware of this issue, not sure how likely this to happen is in practice, it depends on the honesty of validators so you better know.

## Recommended Mitigation Steps
I didn't receive an answer on Discord so decided to submit this FYI to decide if that's a hazard or no.

