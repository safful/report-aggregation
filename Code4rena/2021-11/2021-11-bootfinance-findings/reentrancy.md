## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Reentrancy](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/148) 

# Handle

0v3rf10w


# Vulnerability details

## Impact
Multiple Reentrancy

## Proof of Concept
Reentrancy in BasicSale.receive() (tge/contracts/PublicSale.sol#148-156)

Reentrancy in BasicSale.burnEtherForMember(address) (tge/contracts/PublicSale.sol#158-166)

 State variables written after the external call(s) in all above.

## Tools Used
Manual

## Recommended Mitigation Steps

