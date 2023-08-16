## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [XSS via SVG Construction contract](https://github.com/code-423n4/2022-01-timeswap-findings/issues/131) 

# Handle

thank_you


# Vulnerability details

## Impact
SVG is a unique type of image file format that is often susceptible to Cross-site scripting. If a malicious user is able to inject malicious Javascript into a SVG file, then any user who views the SVG on a website will be susceptible to XSS. This can lead stolen cookies, Denial of Service attacks, and more.

The `NFTTokenURIScaffold` contract generates a SVG via the `NFTSVG.constructSVG` function. One of the arguments used by the `NFTSVG.constructSVG` function is `svgTitle` which represents the ERC20 symbols of both the asset and collateral ERC20 tokens. When generating an ERC20 contract, a malicious user can set malicious XSS as the ERC20 symbol. 

These set of circumstances leads to XSS when the SVG is loaded on any website.


## Proof of Concept
1. Hacker generates an ERC20 token with a symbol that contains malicious Javascript.
2. Hacker generates a TimeSwap Pair with an asset or collateral that matches the malicious ERC20 token created in Step 1.
3. When `NFTTokenURIScaffold#constructTokenURI` is called, a SVG is generated. This process works such that when generating the SVG the tainted ERC20 symbol created in Step 1 is [passed](https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/libraries/NFTTokenURIScaffold.sol#L90) to the `NFTSVG.constructSVG` function [here](https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/libraries/NFTTokenURIScaffold.sol#L102). This function returns a SVG [containing](https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/libraries/NFTSVG.sol#L27) the tainted ERC20 symbol.
4. When the SVG is loaded on any site such as OpenSea, any user viewing that SVG will load the malicious Javascript from within the SVG and result in a XSS attack.


## Tools Used
N/A

## Recommended Mitigation Steps
Creating a SVG file inside of a Solidity contract is novel and thus requires the entity creating a SVG file to sanitize any potential user-input that goes into generating the SVG file. 

As of this time there are no known Solidity libraries that sanitize text to prevent an XSS attack. The easiest solution is to remove all user-input data from the SVG file or not generate the SVG at all.


