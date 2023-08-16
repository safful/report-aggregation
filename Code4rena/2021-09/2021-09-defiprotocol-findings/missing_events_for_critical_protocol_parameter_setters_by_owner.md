## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Missing events for critical protocol parameter setters by owner](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/171) 

# Handle

0xRajeev


# Vulnerability details

## Impact

None of the Factory owner setter functions emit events to record these changes on-chain for off-chain monitors/tools/interfaces to register the updates and react if necessary.

The impact of this is that a malicious/compromised/careless owner can intentionally/accidentally changes the minLicenseFee, auctionDecrement, auctionMultiplier, bondPercentDiv or ownerSplit values that significantly change the security/financial posture/perception of the protocol. No events are emitted and users may lose funds/confidence without being a chance to exit/engage protocol. The protocol takes a reputation hit.


See similar high-severity finding in OpenZeppelin’s Audit of Audius (https://blog.openzeppelin.com/audius-contracts-audit/#high) and medium-severity finding OpenZeppelin’s Audit of UMA Phase 4: https://blog.openzeppelin.com/uma-audit-phase-4/.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L39-L41

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L43-L45

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L47-L49

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L51-L53

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L55-L59


## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Recommend to consider emitting events when protocol critical values are updated by owner. This will be more transparent and it will make it easier to keep track of the status of the system.

