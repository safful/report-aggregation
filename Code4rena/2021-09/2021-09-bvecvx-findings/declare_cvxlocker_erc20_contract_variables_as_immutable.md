## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Declare CvxLocker erc20 contract variables as immutable](https://github.com/code-423n4/2021-09-bvecvx-findings/issues/14) 

# Handle

patitonar


# Vulnerability details

## Impact
Gas optimization to store variables as immutable instead of storage similar to `_decimals`

## Proof of Concept
https://github.com/code-423n4/2021-09-bvecvx/blob/1d64bd58c7a4224cc330cef283561e90ae6a3cf5/veCVX/contracts/locker/CvxLocker.sol#L112-L114

## Tools Used
Manual review

## Recommended Mitigation Steps
Declare as following:
string private immutable _name;
string private immutable _symbol;


