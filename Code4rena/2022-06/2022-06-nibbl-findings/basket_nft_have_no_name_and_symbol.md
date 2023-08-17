## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Basket NFT have no name and symbol](https://github.com/code-423n4/2022-06-nibbl-findings/issues/317) 

# Lines of code

https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/Basket.sol#L13
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/Basket.sol#L6


# Vulnerability details

## Impact
The `Basket` contract is intended to be used behind a proxy. But the `ERC721` implementation used is not upgradeable, and its constructor is called at deployment time on the implementation. So all proxies will have a void name and symbol, breaking all potential integrations and listings.

## Proof of Concept
`ERC721("NFT Basket", "NFTB")` is called at deployment time, and sets private variable at the implementation level. Therefore when loading the code during `delegateCall`, these variables will not be initialized.

## Recommended Mitigation Steps
The easiest mitigation would be to pass this variable as immutable so they are hardcoded in the implementation byte code.

