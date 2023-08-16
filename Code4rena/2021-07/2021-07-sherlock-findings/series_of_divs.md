## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [series of divs](https://github.com/code-423n4/2021-07-sherlock-findings/issues/24) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function payout contains an expression with 3 sequential divs.
This is generally not recommended because it could lead to rounding errors / loss of precision.
Also a div is usually more expensive than a mul.
Also an intermediate division by 0 (if SherXERC20Storage.sx20().totalSupply) == 0) could occur.

## Proof of Concept
//https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/Payout.sol#L108
function payout(
..
 uint256 deduction =  excludeUsd.div(curTotalUsdPool.div(SherXERC20Storage.sx20().totalSupply)).div(10e17);

## Tools Used

## Recommended Mitigation Steps
Verify the formula and replace with something like:
uint256 deduction =  excludeUsd.mul(SherXERC20Storage.sx20().totalSupply).div(  curTotalUsdPool.mul(10e17) )

