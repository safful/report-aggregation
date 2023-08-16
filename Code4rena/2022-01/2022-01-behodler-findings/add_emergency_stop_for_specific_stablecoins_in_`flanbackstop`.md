## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Add emergency stop for specific stablecoins in `FlanBackstop`](https://github.com/code-423n4/2022-01-behodler-findings/issues/88) 

# Handle

Ruhum


# Vulnerability details

## Impact
The recent events concerning MIM showed that stablecoins are not always worth $1. It might be worth it to add an option to stop accepting a specific stablecoin for the time being in `FlanBackstop`.

https://coinmarketcap.com/currencies/magic-internet-money/

Generally, it would allow someone to mint `PyroFlan` for cheaper than expected. Whether there are more possible attack vectors is not entirely clear to me.

I'd argue that you don't lose much by adding it.

## Proof of Concept
Currently, a backer can only be updated through a proposal which will most likely take too long: https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/FlanBackstop.sol#L63

## Tools Used
none

## Recommended Mitigation Steps
Allow pausing the use of specific backers. Using the `governanceApproved()` modifier might be good

