## Tags

- bug
- disagree with severity
- 2 (Med Risk)
- sponsor confirmed

# [changeDAO should be a two-step process in Vader.sol](https://github.com/code-423n4/2021-04-vader-findings/issues/162) 

# Handle

0xRajeev


# Vulnerability details

## Impact

changeDAO() updates DAO address in one-step. If an incorrect address is mistakenly used (and voted upon) then future administrative access or recovering from this mistake is prevented because onlyDAO modifier is used for changeDAO(), which requires msg.sender to be the incorrectly used DAO address (for which private keys may not be available to sign transactions).

Reference: See finding #6 from Trail of Bits audit of Hermez Network: https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf

## Proof of Concept

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L192-L196


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use a two-step process where the old DAO address first proposes new ownership in one transaction and a second transaction from the newly proposed DAO address accepts ownership. A mistake in the first step can be recovered by granting with a new correct address again before the new DAO address accepts ownership. Ideally, there should also be a timelock enforced before the new DAO takes effect.


