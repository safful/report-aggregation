## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Unable to transfer WrappedIbbtc if Oracle go down](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/20) 

# Handle

gzeon


# Vulnerability details

## Impact
In WrappedIbbtc, user will not be able to transfer if oracle.pricePerShare() (L124) revert. This is because balanceToShares() is called in both transfer and transferFrom, which included a call to pricePerShare(). 

If this is the expected behavior, note that WrappedIbbtcEth is behaving the opposite as it use the cached value in a local variable pricePerShare which is only updated upon call to updatePricePerShare().

## Recommended Mitigation Steps
Depending on the specification, one of them need to be changed.

