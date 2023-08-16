## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [implicit underflows](https://github.com/code-423n4/2021-06-gro-findings/issues/6) 

# Handle

gpersoon


# Vulnerability details

## Impact
There are a few underflows that are converted via a typecast afterwards to the expected value. If solidity 0.8.x would be used, then the code would revert.
int256(a-b)  where a and b are uint, For example if a=1 and b=2 then the intermediate result would be uint(-1) == 2**256-1
int256(-x) where x is a uint. For example if x=1 then the intermediate result would be uint(-1) == 2**256-1
Its better not to have underflows by using the appropriate typecasts.
This is especially relevant when moving to solidity 0.8.x

## Proof of Concept
// https://github.com/code-423n4/2021-06-gro/blob/main/contracts/insurance/Exposure.sol#L178
function sortVaultsByDelta(..)
..
        for (uint256 i = 0; i < N_COINS; i++) {
            // Get difference between vault current assets and vault target
            int256 delta = int256(unifiedAssets[i] - unifiedTotalAssets.mul(targetPercents[i]).div(PERCENTAGE_DECIMAL_FACTOR)); // underflow in intermediate result

//https://github.com/code-423n4/2021-06-gro/blob/main/contracts/pnl/PnL.sol#L112
 function decreaseGTokenLastAmount(bool pwrd, uint256 dollarAmount, uint256 bonus)...
..
 emit LogNewGtokenChange(pwrd, int256(-dollarAmount)); // underflow in intermediate result

// https://github.com/code-423n4/2021-06-gro/blob/main/contracts/pools/oracle/Buoy3Pool.sol#L87
 function safetyCheck() external view override returns (bool) {
      ...
            _ratio = abs(int256(_ratio - lastRatio[i])); // underflow in intermediate result
  
## Tools Used

## Recommended Mitigation Steps
replace int256(a-b) with int256(a)-int256(b)
replace int256(-x)   with -int256(x)


