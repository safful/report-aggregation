## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Oracle price does not compound](https://github.com/code-423n4/2022-03-volt-findings/issues/22) 

# Lines of code

https://github.com/code-423n4/2022-03-volt/blob/f1210bf3151095e4d371c9e9d7682d9031860bbd/contracts/oracle/ScalingPriceOracle.sol#L136
https://github.com/code-423n4/2022-03-volt/blob/f1210bf3151095e4d371c9e9d7682d9031860bbd/contracts/oracle/ScalingPriceOracle.sol#L113


# Vulnerability details

## Impact
The oracle does not correctly compound the monthly APRs - it resets on `fulfill`.
Note that the [`oraclePrice` storage variable](https://github.com/code-423n4/2022-03-volt/blob/f1210bf3151095e4d371c9e9d7682d9031860bbd/contracts/oracle/ScalingPriceOracle.sol#L198) is only set in `_updateCPIData` as part of the oracle `fulfill` callback.
It's set to the old price (price from 1 month ago) plus the interpolation from **`startTime`** to now.
However, `startTime` is **reset** in `requestCPIData` due to the `afterTimeInit` modifier, and therefore when Chainlink calls `fulfill` in response to the CPI request, the `timeDelta = block.timestamp - startTime` is close to zero again and `oraclePrice` is updated to itself again.

This breaks the core functionality of the protocol as the oracle does not track the CPI, it always resets to `1.0` after every `fulfill` instead of compounding it.
In addition, there should also be a way for an attacker to profit from the sudden drop of the oracle price to `1.0` again.

#### POC
As an example, assume `oraclePrice = 1.0 (1e18)`, `monthlyAPR = 10%`. The time elapsed is 14 days. Calling `getCurrentOraclePrice()` now would return `1.0 + 14/28 * 10% = 1.05`.

- it's now the 15th of the month and one can trigger `requestCPIData`. **This resets `startTime = now`**.
- Calling `getCurrentOraclePrice()` now would return `1.0` again as `timeDelta` (and `priceDelta`) is zero: `oraclePriceInt + priceDelta = oraclePriceInt = 1.0`.
- When `fulfill` is called it sets `oraclePrice = getCurrentOraclePrice()` which will be close to `1.0` as the `timeDelta` is tiny.

## Recommended Mitigation Steps
The `oraclePrice` should be updated in `requestCPIData()` not in `fulfill`.
Cover this scenario of multi-month accumulation in tests.


