## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Max value of upperStabilityThreshold and lowerStabilityThreshold not checked](https://github.com/code-423n4/2021-11-malt-findings/issues/192) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function setStabilityThresholds of StabilizerNode.sol set the values for upperStabilityThreshold and lowerStabilityThreshold,
however there is no check for a maximum value.
This means that in function _shouldAdjustSupply() the values for upperThreshold and lowerThreshold  could get larger than priceTarget.
When they are subtracted from priceTarget a revert will occur.

Thus it is useful the make sure that upperStabilityThreshold and lowerStabilityThreshold don't get too large.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/StabilizerNode.sol#L445-L454

```JS
function setStabilityThresholds(uint256 _upper, uint256 _lower) external onlyRole(ADMIN_ROLE, "Must have admin role") {
    require(_upper > 0 && _lower > 0, "Must be above 0");
    upperStabilityThreshold = _upper;
    lowerStabilityThreshold = _lower;
```

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/StabilizerNode.sol#L198-L206

```JS
function _shouldAdjustSupply(uint256 exchangeRate) internal view returns (bool) {
   ...
    uint256 upperThreshold = priceTarget.mul(upperStabilityThreshold).div(10**decimals); // upperStabilityThreshold could be > 10**dec => upperThreshold could be > priceTarget
    uint256 lowerThreshold = priceTarget.mul(lowerStabilityThreshold).div(10**decimals);  // lowerStabilityThreshold could be > 10**dec => lowerThreshold could be > priceTarget

    return (exchangeRate <= priceTarget.sub(lowerThreshold) && !auction.auctionActive(auction.currentAuctionId())) || exchangeRate >= priceTarget.add(upperThreshold); // can revert
  }
```

## Tools Used

## Recommended Mitigation Steps
In function setStabilityThresholds() check for a maximum value of upperStabilityThreshold and lowerStabilityThreshold

