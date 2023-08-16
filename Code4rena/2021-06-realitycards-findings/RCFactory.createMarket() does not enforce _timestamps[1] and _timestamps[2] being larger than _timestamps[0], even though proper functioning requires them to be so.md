## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [RCFactory.createMarket() does not enforce _timestamps[1] and _timestamps[2] being larger than _timestamps[0], even though proper functioning requires them to be so](https://github.com/code-423n4/2021-06-realitycards-findings/issues/61) 

# Handle

jvaqa


# Vulnerability details

RCFactory.createMarket() does not enforce _timestamps[1] and _timestamps[2] being larger than _timestamps[0], even though proper functioning requires them to be so.

## Impact

IRCMarket defines a sequence of events that each market should progress through sequentially, CLOSED, OPEN, LOCKED, WITHDRAW. // [1]

The comments explicitly state that _incrementState() should be called "thrice". // [2]

However, it is possible to create a market where these events do not occur sequentially.

You can create a market where the marketOpeningTime is later than the marketLockingTime and oracleResolutionTime.

This is because although RCFactory checks to ensure that _timestamps[2] is greater than _timestamps[1], it does not check to ensure that _timestamps[1] is greater than _timestamps[0]. // [3]

This is also because although RCFactory checks to ensure that _timestamps[0] is equal to or greater than block.timestamp, it makes no check for a minimum value for _timestamps[1] or _timestamps[2], or a relative check between the value of _timestamps[0] and _timestamps[1]. // [4]

Thus, you can create a market where the marketLockingTime and the oracleResolutionTime occur before the marketOpeningTime.

## Proof of Concept

When calling RCFactory.createMarket(), Alice can supply 0 as the argument for _timestamps[1] and _timestamps[2], and any value equal to or greater than block.timestamp for _timestamps[0]. // [5]

## Recommended Mitigation Steps

Add the following check to RCFactory.createMarket():

require(
    _timestamps[0] < _timestamps[1],
    "market must begin before market can lock"
);

[1] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/interfaces/IRCMarket.sol#L7

[2] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L1093

[3] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCFactory.sol#L539

[4] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCFactory.sol#L521

[5] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCFactory.sol#L468

