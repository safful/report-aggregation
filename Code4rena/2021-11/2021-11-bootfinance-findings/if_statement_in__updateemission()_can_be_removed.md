## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [If statement in _updateEmission() can be removed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/34) 

# Handle

Reigada


# Vulnerability details

## Impact
The if statement in _updateEmission() can be removed as the condition is already checked in updateEmission()
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L596
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L186

## Proof of Concept
    function _updateEmission() private {
        if (block.timestamp >= startEpochTime + RATE_TIME) {
            miningEpoch += 1;
            startEpochTime = startEpochTime.add(RATE_TIME);
            startEpochSupply = startEpochSupply.add(rate.mul(RATE_TIME));

            if (miningEpoch < INITIAL_RATE_EPOCH_CUTTOF) {
                rate = rate.mul(EPOCH_INFLATION).div(100000);
            }
            else {
                rate = 0;
            }
            emit updateMiningParameters(block.timestamp, rate, startEpochSupply);
        }
    }

    //Update emission to be called at every step change to update emission inflation
    function updateEmission() public {
        require(block.timestamp >= startEpochTime + RATE_TIME, "Too soon"); // Condition already checked here
        _updateEmission();
    }

## Tools Used
Manual testing

## Recommended Mitigation Steps
Remove the if condition in the _updateEmission() private function

