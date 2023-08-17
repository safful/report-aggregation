## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- selected-for-report

# [NFT transferring won't work because of the external call to `removeDelegation`.](https://github.com/code-423n4/2022-07-golom-findings/issues/377) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowDelegation.sol#L242
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowDelegation.sol#L211


# Vulnerability details

## Impact
The `VoteEscrowDelegation._transferFrom` function won't work because it calls `this.removeDelegation(_tokenId)`. The `removeDelegation` function is external, so when the call is done by `this.removeDelegation(_tokenId)` msg.sender changes to the contract address.

This causes the check in the `` function to (most likely) fail because the contract is not the owner of the NFT, and that will make the function revert.
`require(ownerOf(tokenId) == msg.sender, 'VEDelegation: Not allowed');`

## Tools Used
Manual audit (VS Code & my mind)

## Recommended Mitigation Steps
Make the `removeDelegation` function public and call it without changing the context (i.e. without changing msg.sender to the contract's address).
