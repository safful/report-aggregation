## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Lack of 0 amount check allows malicious user to create infinite vaults](https://github.com/code-423n4/2022-05-cally-findings/issues/91) 

# Lines of code

https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L200


# Vulnerability details

## Impact
A griefer is able to create as many vaults as they want by simply calling `createVault()` with `tokenIdOrAmount = 0`. This will most likely pose problems on the front-end of the Cally protocol because there will be a ridiculously high number of malicious vaults displayed to actual users.

I define these vaults as malicious because it is possible that a user accidently buys a call on this vault which provides 0 value in return. Overall, the presence of zero-amount vaults is damaging to Cally's product image and functionality.

## Proof of Concept
- User calls `createVault(0,,,,);` with an ERC20 type.
- There is no validation that `amount > 0`
- Function will complete successfully, granting the new vault NFT to the caller.
- Cally protocol is filled with unwanted 0 amount vaults.

## Tools Used
Manual review

## Recommended Mitigation Steps
Add the simple check `require(tokenIdOrAmount > 0, "Amount must be greater than 0");`

