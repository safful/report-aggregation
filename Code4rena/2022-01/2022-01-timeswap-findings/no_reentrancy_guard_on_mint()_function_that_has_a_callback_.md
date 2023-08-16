## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [no reentrancy guard on mint() function that has a callback ](https://github.com/code-423n4/2022-01-timeswap-findings/issues/43) 

# Handle

jayjonah8


# Vulnerability details

## Impact
In CollateralizedDebt.sol, the mint() function calls _safeMint() which has a callback to the "to" address argument.  Functions with callbacks should have reentrancy guards in place for protection against possible malicious actors both from inside and outside the protocol.  

## Proof of Concept
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/CollateralizedDebt.sol#L76

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L263

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L395

## Tools Used
Manual code review 

## Recommended Mitigation Steps
Add a reentrancy guard modifier on the mint() function in CollateralizedDebt.sol 

