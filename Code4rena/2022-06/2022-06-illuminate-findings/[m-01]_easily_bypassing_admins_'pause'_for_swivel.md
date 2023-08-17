## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [[M-01] Easily bypassing admins 'pause' for swivel](https://github.com/code-423n4/2022-06-illuminate-findings/issues/343) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L247-L305


# Vulnerability details

## Impact
Assuming admin decides to pause an external principle when it's dangerous, malicious or unprofitable,
Bypassing the admins decision can result in loss of funds for the project.

## Proof of Concept
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L247-L305

* The principals enum `p` is only used for `unpaused(p)` modifier, and to emit an event.
* Attacker can bypass the `unpaused(p)` modifier check by simply passing an enum of another principle that is not paused.
* The function will just continue as normal, without any other side-effect, as if the `pause` is simple ignored.

## Recommended Mitigation Steps
Add this check at the beginning of the function (just like in similar functions of this solution)
`        if (p != uint8(MarketPlace.Principals.Swivel)) {
            revert Invalid('principal');
        }
`

