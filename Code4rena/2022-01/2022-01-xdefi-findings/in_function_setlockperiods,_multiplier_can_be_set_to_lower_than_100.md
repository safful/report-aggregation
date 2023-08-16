## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [in function setLockPeriods, multiplier can be set to lower than 100](https://github.com/code-423n4/2022-01-xdefi-findings/issues/96) 

# Handle

Tomio


# Vulnerability details

## Impact
in function setLockPeriods multiplier can be set to lower than 100 which will break the calculation when dividing the multiplier in function _lock https://github.com/XDeFi-tech/xdefi-distribution/blob/master/contracts/XDEFIDistribution.sol#L268. If the amount times bonus multiplier below 100 the units value will be 0, therefore the totalUnits won't be added but the positionOf[tokenId_] bill be added.

## Proof of Concept
https://github.com/XDeFi-tech/xdefi-distribution/blob/master/contracts/XDEFIDistribution.sol#L77
https://github.com/XDeFi-tech/xdefi-distribution/blob/master/contracts/XDEFIDistribution.sol#L268

## Tools Used

## Recommended Mitigation Steps
in function setLockPeriods need to be add 
```function setLockPeriods(uint256[] memory durations_, uint8[] memory multipliers) external onlyOwner {   
        uint256 count = durations_.length;

        for (uint256 i; i < count; ++i) {
            require(multipliers >= 100); //added
            uint256 duration = durations_[i];
            require(duration <= uint256(18250 days), "INVALID_DURATION");
            emit LockPeriodSet(duration, bonusMultiplierOf[duration] = multipliers[i]);
        }
    }
```

