## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [sortVaultsByDelta doesn't work as expected](https://github.com/code-423n4/2021-06-gro-findings/issues/2) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function sortVaultsByDelta doesn't always work as expected.
Suppose all the delta's are positive, and delta1 >= delta2 >= delta3 > 0
Then maxIndex = 0
And (delta < minDelta (==0) ) is never true, so minIndex = 0

Then (assuming bigFirst==true):
vaultIndexes[0] = maxIndex = 0
vaultIndexes[2] = minIndex = 0
vaultIndexes[1] = N_COINS - maxIndex - minIndex = 3-0-0 = 3

This is clearly not what is wanted, all vaultIndexes should be different and should be in the range [0..2]
This is due to the fact that maxDelta and minDelta are initialized with the value 0.
This all could results in withdrawing from the wrong vaults and reverts (because vaultIndexes[1]  is out of range).

## Proof of Concept
// https://github.com/code-423n4/2021-06-gro/blob/main/contracts/insurance/Exposure.sol#L178
function sortVaultsByDelta(bool bigFirst,uint256 unifiedTotalAssets,uint256[N_COINS] calldata unifiedAssets,uint256[N_COINS] calldata targetPercents) external pure override returns (uint256[N_COINS] memory vaultIndexes) {
        uint256 maxIndex;
        uint256 minIndex;
        int256 maxDelta;
        int256 minDelta;
        for (uint256 i = 0; i < N_COINS; i++) {
            // Get difference between vault current assets and vault target
            int256 delta = int256(
                unifiedAssets[i] - unifiedTotalAssets.mul(targetPercents[i]).div(PERCENTAGE_DECIMAL_FACTOR)
            );
            // Establish order
            if (delta > maxDelta) {
                maxDelta = delta;
                maxIndex = i;
            } else if (delta < minDelta) {
                minDelta = delta;
                minIndex = i;
            }
        }
        if (bigFirst) {
            vaultIndexes[0] = maxIndex;
            vaultIndexes[2] = minIndex;
        } else {
            vaultIndexes[0] = minIndex;
            vaultIndexes[2] = maxIndex;
        }
        vaultIndexes[1] = N_COINS - maxIndex - minIndex;
    }

## Tools Used

## Recommended Mitigation Steps
Initialize maxDelta and minDelta:
        int256 maxDelta = -2**255; // or type(int256).min when using a newer solidity version
        int256 minDelta  = 2**255; // or type(int256).max when using a newer solidity version
Check maxIndex and minIndex are not the same
require (maxIndex != minIndex);

