## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [SynthVault withdraw forfeits rewards](https://github.com/code-423n4/2021-07-spartan-findings/issues/168) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details

The `SynthVault.withdraw` function does not claim the user's rewards.
It decreases the user's weight and therefore they are forfeiting their accumulated rewards.
The `synthReward` variable in `_processWithdraw` is also never used - it was probably intended that this variable captures the claimed rewards.

## Impact
Usually, withdrawal functions claim rewards first but this one does not.
A user that withdraws loses all their accumulated rewards.

## Recommended Mitigation Steps
Claim the rewards with the user's deposited balance first in `withdraw`.


