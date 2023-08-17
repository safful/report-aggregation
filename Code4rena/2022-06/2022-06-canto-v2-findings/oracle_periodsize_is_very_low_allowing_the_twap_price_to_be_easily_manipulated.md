## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Oracle periodSize is very low allowing the TWAP price to be easily manipulated](https://github.com/code-423n4/2022-06-canto-v2-findings/issues/124) 

# Lines of code

https://github.com/Plex-Engineer/lending-market-v2/blob/ea5840de72eab58bec837bb51986ac73712fcfde/contracts/Stableswap/BaseV1-core.sol#L72


# Vulnerability details

## Impact
TWAP oracle easily manipulated

## Proof of Concept
periodSize is set to 0 meaning that the oracle will take a new observation every single block, which would allow an attacker to easily flood the TWAP oracle and manipulate the price

## Tools Used

## Recommended Mitigation Steps
Increase periodSize to be greater than 0, 1800 is typically standard

