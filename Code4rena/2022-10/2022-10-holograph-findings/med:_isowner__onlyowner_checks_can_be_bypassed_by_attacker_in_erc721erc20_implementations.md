## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- responded

# [MED: isOwner / onlyOwner checks can be bypassed by attacker in ERC721/ERC20 implementations](https://github.com/code-423n4/2022-10-holograph-findings/issues/464) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/abstract/ERC721H.sol#L185
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/abstract/ERC721H.sol#L121


# Vulnerability details

## Description

ERC20H and ERC721H are base contracts for NFTs / coins to inherit from. They supply the modifier onlyOwner and function isOwner which are used in the implementations for access control. However, there are several functions which when using these the answer may be corrupted to true by an attacker.

The issue comes from confusion between calls coming from HolographERC721's fallback function, and calls from actually implemented functions. 

In the fallback function, the enforcer appends an additional 32 bytes of msg.sender :
```
assembly {
  calldatacopy(0, 0, calldatasize())
  mstore(calldatasize(), caller())
  let result := call(gas(), sload(_sourceContractSlot), callvalue(), 0, add(calldatasize(), 32), 0, 0)
  returndatacopy(0, 0, returndatasize())
  switch result
  case 0 {
    revert(0, returndatasize())
  }
  default {
    return(0, returndatasize())
  }
}
```

Indeed these are the bytes read as msgSender:
```
function msgSender() internal pure returns (address sender) {
  assembly {
    sender := calldataload(sub(calldatasize(), 0x20))
  }
}
```

and isOwner simply compares these to the stored owner:
```
function isOwner() external view returns (bool) {
  if (msg.sender == holographer()) {
    return msgSender() == _getOwner();
  } else {
    return msg.sender == _getOwner();
  }
}
```

However, the enforcer calls these functions directly in several locations, and in these cases it of course does not append a 32 byte msg.sender. For example, in safeTransferFrom:
```
function safeTransferFrom(
  address from,
  address to,
  uint256 tokenId,
  bytes memory data
) public payable {
  require(_isApproved(msg.sender, tokenId), "ERC721: not approved sender");
  if (_isEventRegistered(HolographERC721Event.beforeSafeTransfer)) {
    require(SourceERC721().beforeSafeTransfer(from, to, tokenId, data));
  }
  _transferFrom(from, to, tokenId);
  if (_isContract(to)) {
    require(
      (ERC165(to).supportsInterface(ERC165.supportsInterface.selector) &&
        ERC165(to).supportsInterface(ERC721TokenReceiver.onERC721Received.selector) &&
        ERC721TokenReceiver(to).onERC721Received(address(this), from, tokenId, data) ==
        ERC721TokenReceiver.onERC721Received.selector),
      "ERC721: onERC721Received fail"
    );
  }
  if (_isEventRegistered(HolographERC721Event.afterSafeTransfer)) {
    require(SourceERC721().afterSafeTransfer(from, to, tokenId, data));
  }
}
```

Here, caller has arbitrary control of the data parameter, and can pass owner's address.When the implementation, SourceERC721(), gets called, beforeSafeTransfer / afterSafeTransfer will behave as if they are called by owner.

Therefore, depending on the actual implementation, derived contracts can lose funds by specifying owner-specific logic. 

This pattern occurs with the following functions, which have an arbitrary data parameter:
- beforeSafeTransfer / after SafeTransfer
- beforeTransfer / afterTransfer
- beforeOnERC721Received / afterOnERC721Received
- beforeOnERC20Received / aferERC20Received

## Impact

Owner-specific functionality can be initiated on NFT / ERC20 implementation contracts

## Tools Used

Manual audit

## Recommended Mitigation Steps

Refactor the code to represent msg.sender information in a bug-free way.