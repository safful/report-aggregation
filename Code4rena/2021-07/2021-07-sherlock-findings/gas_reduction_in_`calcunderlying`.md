## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [gas reduction in `calcUnderlying`](https://github.com/code-423n4/2021-07-sherlock-findings/issues/124) 

# Handle

0xsanson


# Vulnerability details

## Impact
In the `calcUnderlying` function of LibSherX.sol, the value `gs.tokensSherX.length` can be written down once to save gas (around 300-500 when called in the present tests).

## Proof of Concept
https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/libraries/LibSherX.sol#L55-L60

## Tools Used
hardhat gas calculator

## Recommended Mitigation Steps
Suggested adding `uint256 SherXLength = gs.tokensSherX.length;` and replacing this value throughout the function (three instances).

