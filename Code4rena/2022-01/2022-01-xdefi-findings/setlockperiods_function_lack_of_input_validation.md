## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [setLockPeriods function lack of input validation](https://github.com/code-423n4/2022-01-xdefi-findings/issues/38) 

# Handle

cccz


# Vulnerability details

## Impact

In the setLockPeriods function, there is no verification of the multipliers parameter, multipliers[i] may be 0, and the length of multipliers may not be equal to the length of durations_.

```
    function setLockPeriods(uint256[] memory durations_, uint8[] memory multipliers) external onlyOwner {
        uint256 count = durations_.length;

        for (uint256 i; i < count; ++i) {
            uint256 duration = durations_[i];
            require(duration <= uint256(18250 days), "INVALID_DURATION");
            emit LockPeriodSet(duration, bonusMultiplierOf[duration] = multipliers[i]);
        }
    }
```

## Proof of Concept

https://github.com/XDeFi-tech/xdefi-distribution/blob/v1.0.0-beta.0/contracts/XDEFIDistribution.sol#L77-L85

## Tools Used

Manual analysis

## Recommended Mitigation Steps

```
    function setLockPeriods(uint256[] memory durations_, uint8[] memory multipliers) external onlyOwner {
+      require(durations_.length == multipliers.length);
        uint256 count = durations_.length;

        for (uint256 i; i < count; ++i) {
            uint256 duration = durations_[i];
+         require(multipliers[i] != 0);
            require(duration <= uint256(18250 days), "INVALID_DURATION");
            emit LockPeriodSet(duration, bonusMultiplierOf[duration] = multipliers[i]);
        }
    }
```



