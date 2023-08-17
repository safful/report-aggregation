## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- responded

# [MED - Incorrect implementation of ERC721 may have bad consequences for receiver](https://github.com/code-423n4/2022-10-holograph-findings/issues/469) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/HolographERC721.sol#L467


# Vulnerability details

## Description

HolographERC721.sol is an enforcer contract that fully implements ERC721. In its safeTransferFromFunction there is the following code:
```
if (_isContract(to)) {
  require(
    (ERC165(to).supportsInterface(ERC165.supportsInterface.selector) &&
      ERC165(to).supportsInterface(ERC721TokenReceiver.onERC721Received.selector) &&
      ERC721TokenReceiver(to).onERC721Received(address(this), from, tokenId, data) ==
      ERC721TokenReceiver.onERC721Received.selector),
    "ERC721: onERC721Received fail"
  );
}
```
If the target address is a contract, the enforcer requires the target's onERC721Received() to succeed. However, the call deviates from the [standard](https://eips.ethereum.org/EIPS/eip-721):
```
interface ERC721TokenReceiver {
    /// @notice Handle the receipt of an NFT
    /// @dev The ERC721 smart contract calls this function on the recipient
    ///  after a `transfer`. This function MAY throw to revert and reject the
    ///  transfer. Return of other than the magic value MUST result in the
    ///  transaction being reverted.
    ///  Note: the contract address is always the message sender.
    /// @param _operator The address which called `safeTransferFrom` function
    /// @param _from The address which previously owned the token
    /// @param _tokenId The NFT identifier which is being transferred
    /// @param _data Additional data with no specified format
    /// @return `bytes4(keccak256("onERC721Received(address,address,uint256,bytes)"))`
    ///  unless throwing
    function onERC721Received(address _operator, address _from, uint256 _tokenId, bytes _data) external returns(bytes4);
}
```

The standard mandates that the first parameter will be the operator - the caller of safeTransferFrom. The enforcer passes instead the address(this) value, in other words the Holographer address. The impact is that any bookkeeping done in target contract, and allow / disallow decision of the transaction, is based on false information.

## Impact

ERC721 transferFrom's "to" contract may fail to accept transfers, or record credit of transfers incorrectly. 

## Tools Used

Manual audit

## Recommended Mitigation Steps

Pass the msg.sender parameter, as the ERC721 standard requires.