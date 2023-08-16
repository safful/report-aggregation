## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Publisher role cannot be renounced](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/185) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Renouncing ownership is desirable in certain scenarios and is typically allowed by libraries such as Ownable.  The same may be true of the publisher role in this protocol as well to prevent changing the license fee or re-indexing the basket forever. 

This is typically done by assigning a zero address to such a role i.e. burning it. However, by requiring any new proposed publisher address to be != zero address, the current implementation does not provide an option to renounce a publisher role by burning it.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L134

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Consider adding support to renounce the publisher role or specify why this is not a desirable requirement for the protocol.

