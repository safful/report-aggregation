## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Use of uint8 for counter in for loop increases gas costs](https://github.com/code-423n4/2021-10-tempus-findings/issues/38) 

# Handle

TomFrench


# Vulnerability details

## Impact
Increased gas costs

## Proof of Concept

On L189, we use a uint8 as the for loop variable:
https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/amm/TempusAMM.sol#L189

Due to how the EVM natively works on 256 numbers, using a 8 bit number here introduces additional costs as the EVM has to properly enforce the limits of this smaller type.

See the warning at this link: https://docs.soliditylang.org/en/v0.8.0/internals/layout_in_storage.html#layout-of-state-variables-in-storage

## Recommended Mitigation Steps

Change i to be a uint256 and replace any similar uints which only exist in memory and aren't required to use a smaller type.

