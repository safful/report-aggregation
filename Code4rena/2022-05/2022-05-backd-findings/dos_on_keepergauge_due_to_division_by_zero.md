## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [DoS on KeeperGauge due to division by zero](https://github.com/code-423n4/2022-05-backd-findings/issues/35) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/KeeperGauge.sol#L151-L164


# Vulnerability details

## Impact
In the **_calcTotalClaimable()** function it should be validated that perPeriodTotalFees[i] != 0 since otherwise it would generate a DoS in **claimableRewards()** and **claimRewards()**.
This would be possible since if **advanceEpoch()** or **kill()** is executed by the InflationManager address, the epoch will go up without perPeriodTotalFees[newIndexEpoch] is 0.
The negative of this is that every time the **InflationManager** executes these two methods (**kill() and advanceEpoch()**) DoS is generated until you run **reportFees()**.
Another possible case is that **kill()** or **advanceEpoch()** are executed 2 times in a row and there is no way of a perPeriodTotalFees[epoch-1] updating its value, therefore it would be an irreversible DoS.

## Recommended Mitigation Steps
Generate a behavior for the case that perPeriodTotalFees[i] == 0.


