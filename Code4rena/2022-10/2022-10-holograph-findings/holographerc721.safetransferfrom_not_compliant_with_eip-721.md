## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- responded

# [HolographERC721.safeTransferFrom not compliant with EIP-721](https://github.com/code-423n4/2022-10-holograph-findings/issues/203) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/24bc4d8dfeb6e4328d2c6291d20553b1d3eff00b/src/enforcer/HolographERC721.sol#L366


# Vulnerability details

## Impact
According to EIP-721, we have the following for `safeTransferFrom`:
```solidity
///  (...) When transfer is complete, this function
///  checks if `_to` is a smart contract (code size > 0). If so, it calls
///  `onERC721Received` on `_to` and throws if the return value is not
///  `bytes4(keccak256("onERC721Received(address,address,uint256,bytes)"))`.
```
According to the specification, the function must therefore always call `onERC721Received`, not only when it has determined via ERC-165 that the contract provides this function. Note that in the EIP, the provided interface for `ERC721TokenReceiver` does not mention ERC-165. For the token itself, we have: `interface ERC721 /* is ERC165 */ {`
However, for the receiver, the provided interface there is just: `interface ERC721TokenReceiver {`
This leads to failed transfers when they should not fail, because many receivers will just implement the `onERC721Received` function (which is sufficient according to the EIP), and not `supportsInterface` for ERC-165 support.

## Proof Of Concept
Let's say a receiver just implements the `IERC721Receiver` from OpenZeppelin: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/IERC721Receiver.sol
Like the provided interface in the EIP itself, this interface does not derive from EIP-165. All of these receivers (which are most receivers in practice) will not be able to receive those tokens, because the `require` statement (that checks for ERC-165 support) reverts.

## Recommended Mitigation Steps
Remove the ERC-165 check in the `require` statement (like OpenZeppelin does: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L436)