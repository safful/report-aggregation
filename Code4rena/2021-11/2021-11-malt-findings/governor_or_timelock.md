## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [governor or timelock](https://github.com/code-423n4/2021-11-malt-findings/issues/187) 

# Handle

gpersoon


# Vulnerability details

## Impact
In the contract Timelock.sol the following onlyRole expression occurs a few times, referring GOVERNER and timelock:
onlyRole(GOVERNOR_ROLE, "Must have timelock role")

Whereas several other onlyRole expressions are referring to governor:
onlyRole(GOVERNOR_ROLE, "Timelock::...: Call must come from governor.")

Either the role should be TIMELOCK_ROLE or the messages should refer consistently to governor.
Otherwise it might be more difficult to solve error messages from reverts.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/Timelock.sol#L68

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/Timelock.sol#L84

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/Timelock.sol#L100

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/Timelock.sol#L115

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/Timelock.sol#L140

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/Timelock.sol#L159

## Tools Used

## Recommended Mitigation Steps
Make the error messages consistent


