## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [SafeERC20.sol is imported but not used in the transferBribes() function ](https://github.com/code-423n4/2022-02-redacted-cartel-findings/issues/4) 

# Lines of code

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/BribeVault.sol#L296


# Vulnerability details

## Impact
In BribeVault.sol the transferBribes() function uses token.transfer() instead of token.safeTransfer. 
Tokens that don’t correctly implement the latest EIP20 spec, like USDT, will be unusable in the protocol as they revert the transaction because of the missing return value.  The fact that the SafeERC20.sol library is imported at the top of the BribeVault.sol implies that safeTransfer should be being used but may have been forgotten.  

## Proof of Concept
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/BribeVault.sol#L296

## Tools Used
Manual code review 

## Recommended Mitigation Steps
It's recommended to use OpenZeppelin’s SafeERC20 versions with the safeTransfer and safeTransferFrom functions that handle the return value check as well as non-standard-compliant tokens.


