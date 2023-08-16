## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [calcProtocolExposureDelta could use a break](https://github.com/code-423n4/2021-06-gro-findings/issues/20) 

# Handle

gpersoon


# Vulnerability details

## Impact
calcProtocolExposureDelta should probably stop executing once it has found the first occurrence where exposure > threshold.
(as is also indicated in the comment).

The current code also works (due to the check protocolExposedDeltaUsd == 0), however inserting a break statement at the end of the "if" is more logical and saves a bit of gas.

## Proof of Concept
//https://github.com/code-423n4/2021-06-gro/blob/main/contracts/insurance/Allocation.sol#L286
///     By defenition, only one protocol can exceed exposure in the current setup.
...
 function calcProtocolExposureDelta(uint256[] memory protocolExposure, SystemState memory sysState) private pure
       returns (uint256 protocolExposedDeltaUsd, uint256 protocolExposedIndex)
    {
        for (uint256 i = 0; i < protocolExposure.length; i++) {
            // If the exposure is greater than the rebalance threshold...
            if (protocolExposedDeltaUsd == 0 && protocolExposure[i] > sysState.rebalanceThreshold) {
                // ...Calculate the delta between exposure and target
                uint256 target = sysState.rebalanceThreshold.sub(sysState.targetBuffer);
                protocolExposedDeltaUsd = protocolExposure[i].sub(target).mul(sysState.totalCurrentAssetsUsd).div(
                    PERCENTAGE_DECIMAL_FACTOR
                );
                protocolExposedIndex = i;   
                 // probably put a break here
            }
        }
    }

## Tools Used

## Recommended Mitigation Steps
Add a break statement at the end of the if

