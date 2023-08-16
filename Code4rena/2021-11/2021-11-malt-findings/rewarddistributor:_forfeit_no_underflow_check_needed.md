## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [RewardDistributor:_forfeit no underflow check needed](https://github.com/code-423n4/2021-11-malt-findings/issues/140) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Require condition checks already underflow condition. There for no underflow check is needed.

- L154: require(forfeited <= _globals.declaredBalance, "Cannot forfeit more than declared");
- L156:_globals.declaredBalance = _globals.declaredBalance.sub(forfeited);

Therefore L156 can be written as: _globals.declaredBalance = _globals.declaredBalance - forfeited;


## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/RewardSystem/RewardDistributor.sol#L153
## Tools Used

## Recommended Mitigation Steps
- L156 can be written as: _globals.declaredBalance = _globals.declaredBalance - forfeited;

