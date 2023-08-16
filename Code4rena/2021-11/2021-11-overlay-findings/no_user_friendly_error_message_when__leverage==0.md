## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [No user friendly error message when _leverage==0](https://github.com/code-423n4/2021-11-overlay-findings/issues/71) 

# Handle

gpersoon


# Vulnerability details

## Impact
Suppose you try to build a position and have set the _leverage accidentally to 0  (which can be done if you call the smart contract directly).
Then the function build() will call enterOI() which will revert when trying to calculate debtAdjusted_ .

However no user friendly error message is given.

## Proof of Concept
https://github.com/code-423n4/2021-11-overlay/blob/914bed22f190ebe7088194453bab08c424c3f70c/contracts/market/OverlayV1Market.sol
```JS
function enterOI ( bool _isLong,  uint _collateral,  uint _leverage ) external onlyCollateral returns (...) {
 ...
        collateralAdjusted_ = _collateral - _impact - fee_;       // will be > 0
        oiAdjusted_ = collateralAdjusted_ * _leverage;            // if _leverage==0 then oiAdjusted_  == 0
        debtAdjusted_ = oiAdjusted_ - collateralAdjusted_;   // will be negative and thus will revert
```
   
## Tools Used

## Recommended Mitigation Steps
Add something like the following to the function build()
require(_leverage != 0, "OVLV1:leverage==0")

