## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved

# [Conviction scoring fails to initialize and bootstrap](https://github.com/code-423n4/2021-05-fairside-findings/issues/26) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Conviction scores for new addresses/users fail to initialize+bootstrap in ERC20ConvictionScore’s _updateConvictionScore() because a new user’s numCheckpoints will be zero and never gets initialized. 

This effectively means that FairSide conviction scoring fails to bootstrap at all, leading to failure of the protocol’s pivotal feature.

## Proof of Concept

When Alice transfers FSD tokens to Bob for the first time, _beforeTokenTransfer(Alice, Bob, 100) is triggered which calls _updateConvictionScore(Bob, 100) on Line55 of ERC20ConvictionScore.sol. 

In function _updateConvictionScore(), given that this is the first time Bob is receiving FSD tokens, numCheckpoints[Bob] will be 0 (Line116) which will make ts = 0 (Line120), and Bob’s FSD balance will also be zero (Bob never has got FSD tokens prior to this) which makes convictionDelta = 0 (Line122) and not let control go past Line129. 

This means that a new checkpoint never gets written, i.e. conviction score never gets initialized, for Bob or for any user for that matter.

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

FairSide’s adjustment of Compound’s conviction scoring is based on time and so needs an initialization to take place vs. Compound’s implementation. A new checkpoint therefore needs to be created+initialized for a new user during token transfer.

