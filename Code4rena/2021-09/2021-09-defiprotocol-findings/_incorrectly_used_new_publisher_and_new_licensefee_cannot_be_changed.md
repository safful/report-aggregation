## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [ Incorrectly used new publisher and new licenseFee cannot be changed](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/186) 

# Handle

0xRajeev


# Vulnerability details

## Impact
A big aspect of a 2-step change, such as done with changePublisher() and changeLicenseFee(), is to allow any incorrectly used new addresses/values to be changed during the timelock period. This requires allowing the newPublisher or newLicenseFee to be a different value from the one used during the earlier approve and resetting the timelock again.

The current implementation only allows setting it once to a non-zero address/value and prevents any such corrections from being made (by checking that the address/value used is the same as that used during the first approve) which enforces the timelock to prevent surprises to users but does not provide the other accident benefits of using a timelock.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L137

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L136-L147

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L155

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L154-L165

## Tools Used
Manual Analysis

## Recommended Mitigation Steps

Recommend adding "&& pendingPublisher.publisher == newPublisher” and "&& pendingLicenseFee.licenseFee == newLicenseFee" to the if conditional predicate expression along with removing of the require() statement for equality check inside the conditional, to allow resetting the pending address/value to a new one if previously used one was incorrect.

