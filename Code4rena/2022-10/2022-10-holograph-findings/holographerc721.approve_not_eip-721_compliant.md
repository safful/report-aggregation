## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- responded

# [HolographERC721.approve not EIP-721 compliant](https://github.com/code-423n4/2022-10-holograph-findings/issues/205) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/24bc4d8dfeb6e4328d2c6291d20553b1d3eff00b/src/enforcer/HolographERC721.sol#L272


# Vulnerability details

## Impact
According to EIP-721, we have for `approve`:
```solidity
///  Throws unless `msg.sender` is the current NFT owner, or an authorized
///  operator of the current owner.
```
An operator in the context of EIP-721 is someone who was approved via `setApprovalForAll`:
```solidity
/// @notice Enable or disable approval for a third party ("operator") to manage
///  all of `msg.sender`'s assets
/// @dev Emits the ApprovalForAll event. The contract MUST allow
///  multiple operators per owner.
/// @param _operator Address to add to the set of authorized operators
/// @param _approved True if the operator is approved, false to revoke approval
function setApprovalForAll(address _operator, bool _approved) external;
```
Besides operators, there are also approved addresses for a token (for which `approve` is used). However, approved addresses can only transfer the token, see for instance the `safeTransferFrom` description:
```solidity
/// @dev Throws unless `msg.sender` is the current owner, an authorized
///  operator, or the approved address for this NFT.
```
`HolographERC721` does not distinguish between authorized operators and approved addresses when it comes to the `approve` function. Because `_isApproved(msg.sender, tokenId)` is used there, an approved address can approve another address, which is a violation of the EIP (only authorized operators should be able to do so).

## Proof Of Concept
Bob calls `approve` to approve Alice on token ID 42 (that is owned by Bob). One week later, Bob sees that a malicious address was approved for his token ID 42 (e.g., because Alice got phished) and stole his token. Bob wonders how this is possible, because Alice should not have the permission to approve other addresses. However, becaue `HolographERC721` did not follow EIP-721, it was possible.

## Recommended Mitigation Steps
Follow the EIP, i.e. do not allow approved addresses to approve other addresses.