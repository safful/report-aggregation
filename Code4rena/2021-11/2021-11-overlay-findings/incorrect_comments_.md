## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Incorrect comments ](https://github.com/code-423n4/2021-11-overlay-findings/issues/49) 

# Handle

ye0lde


# Vulnerability details

## Impact
Code clarity

## Proof of Concept

"@param" should be "@return"
https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/market/OverlayV1Market.sol#L83-L85

Not sure what this comment is for maybe just needs to be deleted.
https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/mothership/OverlayV1Mothership.sol#L155

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Correct the comments if the suggestions are valid.

