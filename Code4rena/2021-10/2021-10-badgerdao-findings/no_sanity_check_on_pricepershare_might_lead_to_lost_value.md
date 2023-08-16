## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [No sanity check on pricePerShare might lead to lost value](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/68) 

# Handle

kenzo


# Vulnerability details

pricePerShare is read either from an oracle or from ibBTC's core.

If one of these is bugged or exploited, there are no safety checks to prevent loss of funds.

## Impact
As pricePerShare is used to calculate transfer amount, a bug or wrong data which returns smaller pricePerShare than it really is could result in drainage of wibbtc from Curve pool.

## Proof of Concept
Curve's swap and remove liquidity functions will both call wibbtc's `transfer` function:
https://etherscan.io/address/0xFbdCA68601f835b27790D98bbb8eC7f05FDEaA9B#code%23L790
https://etherscan.io/address/0xFbdCA68601f835b27790D98bbb8eC7f05FDEaA9B#code%23L831
The `transfer` function calculates the amount to send by calling `balanceToShares`:
https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtcEth.sol#L127
`balanceToShares` calculates the shares (=amount to send) by dividing in `pricePerShare`:
https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtcEth.sol#L156
Therefore, if due to a bug or exploit in ibBTC core / the trusted oracle pricePerShare is smaller than it really is, the amount that will be sent will grow larger. So Curve will send to the user/exploiter doing swap/remove liquidity more tokens that he deserves.

## Tools Used
Manual analysis, hardhat

## Recommended Mitigation Steps
Add sanity check:

pricePerShare should never decrease but only increase with time (as ibbtc accrues interest) (validated with DefiDollar team). This means that on every pricePerShare read/update, if the new pricePerShare is smaller than the current one, we can discard the update as bad data. 

This will prevent an exploiter from draining Curve pool's wibbtc reserves by decreasing pricePerShare.

