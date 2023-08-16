## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [missing whenNotPaused](https://github.com/code-423n4/2022-02-aave-lens-findings/issues/71) 

# Lines of code

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/LensHub.sol#L929


# Vulnerability details

All the external function of LensHub have whenNotPasued modifier.
However, LensHub is erc721 and the transfer function doesn't have the whenNotPaused modifier.

## Impact
In case where the governance wants to stop all activity, they still can't stop transferring profiles nfts.
an example where stopping transferring tokens was actually very helpful:
[https://mobile.twitter.com/flashfish0x/status/1466369783016869892](https://mobile.twitter.com/flashfish0x/status/1466369783016869892)


## Recommended Mitigation Steps
add whenNotPasued to `_beforeTokenTransfer`

