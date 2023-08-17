## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-05

# [`StabilizerNode.stabilize()` should update `lastTracking` as well to avoid an unnecessary incentive.](https://github.com/code-423n4/2023-02-malt-findings/issues/32) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/main/contracts/StabilityPod/StabilizerNode.sol#L163


# Vulnerability details

## Impact
`StabilizerNode.stabilize()` should update `lastTracking` as well to avoid an unnecessary incentive.

Current logic pays unnecessary incentives to track the pool.

## Proof of Concept
`trackPool()` pays an incentive per `trackingBackoff` in order to ensure pool consistency.

```solidity
File: 2023-02-malt\contracts\StabilityPod\StabilizerNode.sol
248:   function trackPool() external onlyActive {
249:     require(block.timestamp >= lastTracking + trackingBackoff, "Too early"); //@audit lastTracking should be updated in stabilize() also
250:     bool success = maltDataLab.trackPool();
251:     require(success, "Too early");
252:     malt.mint(msg.sender, (trackingIncentive * (10**malt.decimals())) / 100); // div 100 because units are cents
253:     lastTracking = block.timestamp;
254:     emit Tracking();
255:   }
```

And `stabilize()` tracks the pool as well and we don't need to pay an incentive unnecessarily in `trackPool()` if `stabilize()` was called recently.

For that, we can update `lastTracking` in `stabilize()`.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Recommend updating `lastTracking` in `stabilize()`.

```solidity
  function stabilize() external nonReentrant onlyEOA onlyActive whenNotPaused {
    // Ensure data consistency
    maltDataLab.trackPool();
    lastTracking = block.timestamp; //++++++++++++++++

    ...
```