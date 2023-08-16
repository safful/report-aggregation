## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing setter function for `OverlayV1Mothership#marginBurnRate`](https://github.com/code-423n4/2021-11-overlay-findings/issues/84) 

# Handle

WatchPug


# Vulnerability details

Based on the context, `marginBurnRate` should be able to be updated after deployment. However, there is no function to update it.

### Recommendation

Change to:

https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/mothership/OverlayV1Mothership.sol#L158-L166

```solidity=158
function adjustGlobalParams(
    uint16 _fee,
    uint16 _feeBurnRate,
    address _feeTo,
    uint _marginBurnRate
) external onlyGovernor {
    fee = _fee;
    feeBurnRate = _feeBurnRate;
    feeTo = _feeTo;
    marginBurnRate = _marginBurnRate;
}
```

Or change `marginBurnRate` to immutable if it's not supposed to be updated later (for gas saving).

