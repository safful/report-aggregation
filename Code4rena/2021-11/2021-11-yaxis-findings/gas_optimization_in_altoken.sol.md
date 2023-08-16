## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas optimization in AlToken.sol](https://github.com/code-423n4/2021-11-yaxis-findings/issues/3) 

# Handle

tqts


# Vulnerability details

## Impact
Double calculation of the same value in mint()

## Proof of Concept
https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/AlToken.sol#L69

## Tools Used
Manual review

## Recommended Mitigation Steps
The _total variable in line 66 is defined as _amount + hasMinted(msg.sender).
Line 69 needs that value again but recalculates it again instead of using the stored one.
Replace line 69 with hasMinted(msg.sender) = _total

