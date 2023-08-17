## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed

# [User rewards would be lost](https://github.com/code-423n4/2022-05-backd-findings/issues/12) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/main/protocol/contracts/tokenomics/AmmGauge.sol#L103


# Vulnerability details

## Impact
Staking is not stopped even when Gauge is killed. User will not be getting any reward for the staked asset.

## Proof of Concept
1. Assume the AMMGauge is killed using kill function (AmmGauge.sol#L49). This sets killed as true

2. poolCheckpoint will not further increase ammStakedIntegral and would simply return false

```
function poolCheckpoint() public virtual override returns (bool) {
        if (killed) {
            return false;
        }
		...
		}
```

3. User calls stakeFor function and is still able to stake amount. 

4. The drawback will be no rewards as poolCheckpoint will only return false and will not update ammStakedIntegral

## Recommended Mitigation Steps
Add below check in stakeFor function, restricting deposit if Gauge is killed

```
require(!killed, "Gauge killed");
```

