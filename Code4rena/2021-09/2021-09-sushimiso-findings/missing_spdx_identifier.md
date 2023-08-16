## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing SPDX Identifier](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/26) 

# Handle

leastwood


# Vulnerability details

## Impact

There are several contracts missing SPDX identifiers which correctly license the contract for open source development:
`MISOAccessFactory.sol`
`MISOAccessControls.sol`
`MISOAdminAccess.sol`
`PointList.sol`
`TokenList.sol`
`MISOMasterChef.sol`
`CalculationsSushiswap.sol`
`MISOHelper.sol`
`PairsHelper.sol`
`USDC.sol`

## Proof of Concept

Refer to listed contracts.

## Tools Used

Compiler warnings

## Recommended Mitigation Steps

Consider adding `// SPDX-License-Identifier: GPL-3.0-only` to the top of the aforementioned files.

