## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [`user.creation` is updated incorrectly when the user tries to extend membership](https://github.com/code-423n4/2021-11-fairside-findings/issues/61) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-fairside/blob/20c68793f48ee2678508b9d3a1bae917c007b712/contracts/network/FSDNetwork.sol#L274-L291

```solidity=274
if (user.creation == 0) {
    user.creation = block.timestamp;
    user.gracePeriod =
        membership[msg.sender].creation +
        MEMBERSHIP_DURATION +
        60 days;
} else {
    uint256 elapsedDurationPercentage = ((block.timestamp -
        user.creation) * 1 ether) / MEMBERSHIP_DURATION;
    if (elapsedDurationPercentage < 1 ether) {
        uint256 durationIncrease = (costShareBenefit.mul(1 ether) /
            (totalCostShareBenefit - costShareBenefit)).mul(
                MEMBERSHIP_DURATION
            ) / 1 ether;
        user.creation += durationIncrease;
        user.gracePeriod += durationIncrease;
    }
}
```

### PoC

1. Alice calls `function purchaseMembership()` and adds 20 ether of `costShareBenefit` on day 1:

```
alice.creation = day 1 timestamp;
alice.gracePeriod = day 791 timestamp;
```

2. Alice calls `function purchaseMembership()` again and adds 20 ether of `costShareBenefit` on day 2:

```
elapsedDurationPercentage = 1/720
durationIncrease = 730 day

alice.creation = day 731 timestamp;
alice.gracePeriod = day 1521 timestamp;
```

Making Alice unable to use any membership features until two years later.

