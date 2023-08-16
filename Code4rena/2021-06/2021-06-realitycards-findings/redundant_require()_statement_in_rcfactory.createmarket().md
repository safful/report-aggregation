## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Redundant require() statement in RCFactory.createMarket()](https://github.com/code-423n4/2021-06-realitycards-findings/issues/62) 

# Handle

jvaqa


# Vulnerability details

Redundant require() statement in RCFactory.createMarket()

## Impact

RCFactory.createMarket() contains two require() statements side-by-side both checking the value of the relative values of _timestamps[0] and block.timestamp. // [1]

However, there is no case where the first require() statement would be triggered without the second require() statement also being triggered, since advancedWarning cannot have a negative value. // [2]

Thus, the first require() statement is redundant, and unnecessarily uses gas.

## Proof of Concept

Alice can call RCFactory.createMarket() with an advancedWarning value greater than zero, and a _timestamps[0] value less than block.timestamp.

## Recommended Mitigation Steps

Remove this require statement:

require(
                _timestamps[0] >= block.timestamp,
                "Market opening time not set"
            ); // [3]


[1] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCFactory.sol#L520

[2] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCFactory.sol#L524

[3] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCFactory.sol#L520

