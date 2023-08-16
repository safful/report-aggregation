## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Packing of state variable ](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/258) 

# Handle

JMukesh


# Vulnerability details

## Impact
bool _iskilled  state variable can be packed with one of the address state variable like {token , owner}  which will save on slot of memory

## Proof of Concept

https://github.com/code-423n4/2021-11-bootfinance/blob/b4ebd0a5ebcbc24f3d15836cdb9759243fc85868/core-contracts/contracts/sol/BTCPoolDelegator.sol#L55

https://github.com/code-423n4/2021-11-bootfinance/blob/b4ebd0a5ebcbc24f3d15836cdb9759243fc85868/core-contracts/contracts/sol/USDPoolDelegator.sol#L51


## Tools Used

manual review

## Recommended Mitigation Steps


