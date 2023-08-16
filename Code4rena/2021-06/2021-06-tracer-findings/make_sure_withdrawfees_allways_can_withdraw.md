## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [make sure withdrawFees allways can withdraw](https://github.com/code-423n4/2021-06-tracer-findings/issues/23) 

# Handle

gpersoon


# Vulnerability details

## Impact
If you call the function withdrawFees and the "tvl" would not be enough for the fee then the code would revert.
In this case the fees cannot be withdrawn.
Although it is unlikely that the tvl would be wrong it is probably better to be able to withdraw the remaining fees.

## Proof of Concept
// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/TracerPerpetualSwaps.sol#L508
function withdrawFees() external override {
        uint256 tempFees = fees;
        fees = 0;
        tvl = tvl - tempFees;

        // Withdraw from the account
        IERC20(tracerQuoteToken).transfer(feeReceiver, tempFees);
        emit FeeWithdrawn(feeReceiver, tempFees);
    }

## Tools Used

## Recommended Mitigation Steps
Add something like:
tempFees = min (fees, tvl);
and change fees=0 to:
fees -= tempFees;

