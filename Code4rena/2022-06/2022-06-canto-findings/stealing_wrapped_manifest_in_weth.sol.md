## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Stealing Wrapped Manifest in WETH.sol](https://github.com/code-423n4/2022-06-canto-findings/issues/19) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/WETH.sol#L85


# Vulnerability details

## Impact
Allows anyone to steal all wrapped manifest from the WETH.sol contract. Attacker can also withdraw
to convert Wrapped Manifest to Manifest.

Issue in approve(address owner, address spender) external function. This allows an attacker to approve themselves to spend another user's tokens.

Attacker can then use transferFrom(address src, address dst, uint wad) function to send tokens to themself.

## Proof of Concept
Hardhat + Chai test to show exploit. Test file is test/POC.js
https://github.com/soosh1337/POC_lending_market_WETH


## Tools Used
VScode, hardhat

## Recommended Mitigation Steps
I believe there is no need for this function. There is another approve(address guy, uint wad) function that uses msg.sender to approve allowance. There should be no need for someone to approve another user's allowance.

Remove the approve(address owner, address spender) function.

