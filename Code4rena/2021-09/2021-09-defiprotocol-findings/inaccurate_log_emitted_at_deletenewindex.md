## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Inaccurate log emitted at deleteNewIndex](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/58) 

# Handle

kenzo


# Vulnerability details

The DeletedNewIndex log emits "publisher", but it might be the auction that called the function.
Note: the event is defined as:
event DeletedNewIndex(address _publisher);
So if you wanted to anyway emit just the publisher, this is not a bug.
However as this function call be called from both publisher and auction, I have a feeling you wanted to emit the msg.sender.

## Impact
Inaccurate data supplied.

## Proof of Concept
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L208

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Emit msg.sender instead of publisher.

