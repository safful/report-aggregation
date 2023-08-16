## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [RewardDistributor:decrementRewards no underflow check needed](https://github.com/code-423n4/2021-11-malt-findings/issues/143) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Require statement conditions checks that no underflow can happen, therefore we don't need to use safe subtraction (underflow check).

- L275:  require(amount <= _globals.declaredBalance, "Can't decrement more than total reward balance");
- L278:  _globals.declaredBalance = _globals.declaredBalance.sub(amount);
=> Rewrite L278 as:  _globals.declaredBalance = _globals.declaredBalance - amount;

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/RewardSystem/RewardDistributor.sol#L271
## Tools Used

## Recommended Mitigation Steps
- rewrite L278 as:  _globals.declaredBalance = _globals.declaredBalance - amount;

