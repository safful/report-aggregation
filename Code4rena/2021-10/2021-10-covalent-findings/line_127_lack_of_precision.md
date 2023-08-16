## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Line 127 lack of precision](https://github.com/code-423n4/2021-10-covalent-findings/issues/36) 

# Handle

pants


# Vulnerability details

The following calculation can be more numeric precise:
uint128 perEpochRateIncrease =uint128(uint256(allocatedTokensPerEpoch)*divider/uint256(totalGlobalShares));

globalExchangeRate += perEpochRateIncrease * (currentEpoch - lastUpdateEpoch);


Change it to:
uint128 perEpochRateIncrease =uint256(allocatedTokensPerEpoch)*divider;
globalExchangeRate += perEpochRateIncrease * (currentEpoch - lastUpdateEpoch) / uint256(totalGlobalShares);

