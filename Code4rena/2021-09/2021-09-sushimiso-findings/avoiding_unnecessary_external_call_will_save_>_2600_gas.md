## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoiding unnecessary external call will save > 2600 gas](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/66) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Market is guaranteed to be finalized by checking and calling finalize if not finalized. So the subsequent require()  by again checking market.finalized() is redundant and can save 2600+ gas by removing the external call. External calls cost 2600 gas after Berlin upgrade.

## Proof of Concept

https://github.com/sushiswap/miso/blob/2cdb1486a55ded55c81898b7be8811cb68cfda9e/contracts/Liquidity/PostAuctionLauncher.sol#L226-L229

## Tools Used
Manual Analysis

## Recommended Mitigation Steps

Remove the require().

