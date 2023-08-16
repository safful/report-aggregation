## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [use of approve() instead of safeApprove()](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/114) 

# Handle

JMukesh


# Vulnerability details

## Impact
by using approve() , we are not checking the value returned by the approve ,wether it got failed or successfully executed. so it is safe to use safeApproval()

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L226

## Tools Used
manual review

## Recommended Mitigation Steps
use safeApprove()

