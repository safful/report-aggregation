## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [TeamLockup releases more tokens that it should](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/263) 

# Handle

kenzo


# Vulnerability details

TeamLockup mentions on "vestingLength" that it is the "number of YETI that are claimable every day after vesting starts". However, the vesting calculation treats it as if was the number of YETI that are claimable every second, not every day.

## Impact
Tokens would be released faster than planned.
Or, if the tokens are planned to be released every second and not every day (I'm guessing it's less likely), then this is a wrong comment.

## Proof of Concept
The description of `vestingLength`: [(Code ref)](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/YETI/TeamLockup.sol#L15)
```
    uint immutable vestingLength; // number of YETI that are claimable every day after vesting starts
```

The calculation to decide how many tokens can be released: [(Code ref)](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/YETI/TeamLockup.sol#L41:#L43)
```
        uint timePastVesting = block.timestamp.sub(vestingStart);
        uint available = _min(totalVest,(totalVest.mul(timePastVesting)).div(vestingLength));
```
The problem is that `timePastVesting` is in seconds, and `vestingLength` is in days.

## Recommended Mitigation Steps
Divide the calculation by `1 day` to align the units.

