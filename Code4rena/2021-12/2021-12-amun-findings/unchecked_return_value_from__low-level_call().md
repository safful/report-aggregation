## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Unchecked return value from  low-level call()](https://github.com/code-423n4/2021-12-amun-findings/issues/237) 

# Handle

JMukesh


# Vulnerability details

## Impact
The return value of the low-level call is not checked, so if the call fails, the Ether will be locked in the contract. If the low level is used to prevent blocking operations, consider logging failed calls.

## Proof of Concept

https://github.com/code-423n4/2021-12-amun/blob/98f6e2ff91f5fcebc0489f5871183566feaec307/contracts/basket/contracts/singleJoinExit/EthSingleTokenJoinV2.sol#L26

## Tools Used

manual review

## Recommended Mitigation Steps

add condition to check return value 

