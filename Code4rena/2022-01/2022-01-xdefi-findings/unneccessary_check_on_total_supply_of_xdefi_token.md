## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unneccessary check on total supply of XDEFI token](https://github.com/code-423n4/2022-01-xdefi-findings/issues/3) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
Extra gas costs of all locking operations.

## Proof of Concept

XDEFIDistribution.sol stores the total supply of the XDEFI token:

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L14

This is so that the amount being locked can be checked to be less than this on each call

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L255

This is unnecessary as the XDEFI token has no external mint function and so has a fixed supply. It's then impossible for any user to supply more than 240M XDEFI in order to fail this check.

https://etherscan.io/address/0x72b886d09c117654ab7da13a14d603001de0b777#code

## Recommended Mitigation Steps

Remove the unnecessary check on the total supply.

