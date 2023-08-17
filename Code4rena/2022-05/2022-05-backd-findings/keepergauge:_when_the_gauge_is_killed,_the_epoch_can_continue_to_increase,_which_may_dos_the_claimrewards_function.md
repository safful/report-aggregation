## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed

# [KeeperGauge: When the Gauge is killed, the epoch can continue to increase, which may DOS the claimRewards function](https://github.com/code-423n4/2022-05-backd-findings/issues/51) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/KeeperGauge.sol#L157-L161


# Vulnerability details

## Impact
When the Gauge is killed, the advanceEpoch and kill functions can still be called to make epoch+1, while the reportFees function cannot be called to update the value of perPeriodTotalFees, which will cause perPeriodTotalFees[epoch] == 0. Later if the user calls the claimRewards function, the default epoch parameter will cause a divide by zero crash in the code below.
````
for (uint256 i = startEpoch; i < endEpoch; i = i.uncheckedInc()) {
             totalClaimable += (
                 keeperRecords[beneficiary].feesInPeriod[i].scaledDiv(perPeriodTotalFees[i])
             ).scaledMul(perPeriodTotalInflation[i]);
         }
````
## Proof of Concept
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/KeeperGauge.sol#L157-L161
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/KeeperGauge.sol#L96-L100
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/KeeperGauge.sol#L57-L62
## Tools Used
None
## Recommended Mitigation Steps
Require killed to be false in poolCheckpoint function
```
    function poolCheckpoint() public override returns (bool) {
-       if (killed) return false;
+      require(!killed);
        uint256 timeElapsed = block.timestamp - uint256(lastUpdated);
        uint256 currentRate = IController(controller).inflationManager().getKeeperRateForPool(pool);
        perPeriodTotalInflation[epoch] += currentRate * timeElapsed;
        lastUpdated = uint48(block.timestamp);
        return true;
    }
```

