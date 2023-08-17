## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed
- selected-for-report

# [ `VoteEscrowCore.safeTransferFrom` does not check correct magic bytes returned from receiver contract's `onERC721Received` function](https://github.com/code-423n4/2022-07-golom-findings/issues/577) 

# Lines of code

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ce0068c21ecd97c6ec8fb0db08570f4b43029dde/contracts/token/ERC721/ERC721.sol#L395-L417


# Vulnerability details

## Impact

While `VoteEscrowCore.safeTransferFrom` does try to call `onERC721Received` on the receiver it does not check the for the required "magic bytes" which is `IERC721.onERC721received.selector` in this case. See [OpenZeppelin docs](https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#IERC721Receiver-onERC721Received-address-address-uint256-bytes-) for more information.

It's quite possible that a call to `onERC721Received` could succeed because the contract had a `fallback` function implemented, but the contract is not ERC721 compliant.

The impact is that NFT tokens may be sent to non-compliant contracts and lost.

## Proof of Concept

[Lines 604 - 605](https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowCore.sol#L604-L605) are:

```solidity
try IERC721Receiver(_to).onERC721Received(msg.sender, _from, _tokenId, _data) returns (bytes4) {} catch (
    bytes memory reason
```

but they should be:

```solidity
try IERC721Receiver(to).onERC721Received(_msgSender(), from, tokenId, data) returns (bytes4 retval) {
    return retval == IERC721Receiver.onERC721Received.selector;
} catch (bytes memory reason)
```

## Recommended Mitigation Steps

Implement `safeTransferReturn` so that it checks the required magic bytes: `IERC721Receiver.onERC721Received.selector`.
