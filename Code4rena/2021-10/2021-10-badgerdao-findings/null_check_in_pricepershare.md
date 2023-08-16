## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Null check in pricePerShare](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/90) 

# Handle

hack3r-0m


# Vulnerability details

https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtcEth.sol#L73

https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtc.sol#L123

oracle can `0` as a price of the share, in that case, 0 will be the denominator in some calculations which can cause reverts from SafeMath (for e.g here: https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtc.sol#L148 ) resulting in Denial Of Service.

Add a null check to ensure that on every update, the price is greater than 0.

