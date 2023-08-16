## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Missing check on `setManualPrice(int256 _price)`](https://github.com/code-423n4/2021-04-maple-findings/issues/85) 

# Handle

@cmichelio


# Vulnerability details

## Vulnerability Details

The `ChainlinkOracle.setManualPrice` function specifies that it can only be called "if manualOverride == true".

This is not the case.


## Impact

Assume an oracle failure happened, and the oracle needs to be manually set to prevent losses.
The `setManualPrice` function succeeds and the owner might think that the oracle price is overwritten as the function would fail when `manualOverride` is not `true` according to specification.
The protocol would still use the broken chainlink price feed and suffer losses.

## Recommended Mitigation Steps

Add the missing `require(manualOverride == true, "manual override not set")` check.


