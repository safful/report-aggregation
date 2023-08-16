## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Use constants for numbers](https://github.com/code-423n4/2021-05-yield-findings/issues/3) 

# Handle

gpersoon


# Vulnerability details

## Impact
In several locations in the code numbers like 1e12, 1e18, 1e27 are used. The same goes for values like: type(uint256).max
It quite easy to make a mistake somewhere, also when comparing values.

## Proof of Concept
.\Cauldron.sol:        (uint256 rateAtMaturity,) = rateOracle.get(series_.baseId, bytes32("rate"), 1e18);
.\Cauldron.sol:            (uint256 rate,) = rateOracle.get(series_.baseId, bytes32("rate"), 1e18);
.\Cauldron.sol:        accrual_ = accrual_ >= 1e18 ? accrual_ : 1e18;     // The accrual can't be below 1 (with 18 decimals)
.\Cauldron.sol:        uint256 ratio = uint256(spotOracle_.ratio) * 1e12;   // Normalized to 18 decimals
.\FYToken.sol:        (_chiAtMaturity,) = oracle.get(underlyingId, CHI, 1e18);
.\FYToken.sol:            (uint256 chi,) = oracle.get(underlyingId, CHI, 1e18);
.\FYToken.sol:        accrual_ = accrual_ >= 1e18 ? accrual_ : 1e18;     // The accrual can't be below 1 (with 18 decimals)
.\Witch.sol:        require (initialProportion_ <= 1e18, "Only at or under 100%");
.\Witch.sol:            uint256 term2 = initialProportion_ + (1e18 - initialProportion_).wmul(dividend2.wdiv(divisor2));
.\Witch.sol:            price = uint256(1e18).wdiv(term1.wmul(term2));
.\oracles\chainlink\ChainlinkMultiOracle.sol:        value = price * amount / 1e18;
.\oracles\chainlink\ChainlinkMultiOracle.sol:        value = price * amount / 1e18;
.\oracles\compound\CompoundMultiOracle.sol:        value = price * amount / 1e18;
.\oracles\compound\CompoundMultiOracle.sol:        value = price * amount / 1e18;
.\yieldspace\Pool.sol:            uint256 scaledFYTokenCached = uint256(_fyTokenCached) * 1e27;
.\yieldspace\YieldMath.sol:      result = result > 1e12 ? result - 1e12 : 0; // Subtract error guard, flooring the result at zero
.\yieldspace\YieldMath.sol:      result = result > 1e12 ? result - 1e12 : 0; // Subtract error guard, flooring the result at zero
.\yieldspace\YieldMath.sol:      result = result < MAX - 1e12 ? result + 1e12 : MAX; // Add error guard, ceiling the result at max
.\yieldspace\YieldMath.sol:      result = result < MAX - 1e12 ? result + 1e12 : MAX; // Add error guard, ceiling the result at max

.\FYToken.sol:    uint256 public chiAtMaturity = type(uint256).max;           // Spot price (exchange rate) between the base and an interest accruing token at maturity
.\FYToken.sol:        require (chiAtMaturity == type(uint256).max, "Already matured");
.\FYToken.sol:        if (chiAtMaturity == type(uint256).max) {  // After maturity, but chi not yet recorded. Let's record it, and accrual is then 1.
 
 ## Tools Used
grep

## Recommended Mitigation Steps
Define constants for the numbers used throughout the code.


