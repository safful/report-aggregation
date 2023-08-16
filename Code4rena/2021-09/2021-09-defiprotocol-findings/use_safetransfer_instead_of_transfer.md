## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Use safeTransfer instead of transfer](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/196) 

# Handle

hack3r-0m


# Vulnerability details

https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Auction.sol#L146


`transfer()` might return false instead of reverting, in this case, ignoring return value leads to considering it successful.

use `safeTransfer()` or check the return value if length of returned data is > 0.

