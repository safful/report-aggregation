## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache storage variables in the stack can save gas](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/289) 

# Handle

WatchPug


# Vulnerability details

For the storage variables that will be accessed multiple times, cache them in the stack can save ~100 gas from each extra read (`SLOAD` after Berlin).

For example:

- `troveManager` in `TroveManagerRedemptions#_updateBaseRateFromRedemption()`

    https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/TroveManagerRedemptions.sol#L432-L447

    ```solidity=432
    function _updateBaseRateFromRedemption(uint256 _YUSDDrawn, uint256 _totalYUSDSupply)
    internal
    returns (uint256)
    {
        uint256 decayedBaseRate = troveManager.calcDecayedBaseRate();

        /* Convert the drawn Collateral back to YUSD at face value rate (1 YUSD:1 USD), in order to get
         * the fraction of total supply that was redeemed at face value. */
        uint256 redeemedYUSDFraction = _YUSDDrawn.mul(10**18).div(_totalYUSDSupply);

        uint256 newBaseRate = decayedBaseRate.add(redeemedYUSDFraction.div(BETA));
        newBaseRate = LiquityMath._min(newBaseRate, DECIMAL_PRECISION); // cap baseRate at a maximum of 100%

        troveManager.updateBaseRate(newBaseRate);
        return newBaseRate;
    }
    ```

