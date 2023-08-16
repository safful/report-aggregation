## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [msgSender() or _msgSender()](https://github.com/code-423n4/2021-08-realitycards-findings/issues/22) 

# Handle

gpersoon


# Vulnerability details

## Impact
The code has two implementations of msgSender: 
-   msgSender() => uses meta transaction signer
-  _msgSender() => maps to msg.sender

_msgSender() is used in a few locations
- when using _setupRole, this seems legitimate
- in function withdraw  (whereas the similar function withdrawWithMetadata uses msgSender() )

It is confusing to have multiple functions with almost the same name, this could easily lead to mistakes.

## Proof of Concept
// https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/lib/NativeMetaTransaction.sol#L105
 function msgSender() internal view returns (address payable sender) {
        if (msg.sender == address(this)) {
            assembly {   sender := shr(96, calldataload(sub(calldatasize(), 20)))   }
        } else {
             sender = payable(msg.sender);
        }
        return sender;
    }

// https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Context.sol
  function _msgSender() internal view virtual returns (address) {
        return msg.sender;
    }

//https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/nfthubs/RCNftHubL2.sol#L164
  function withdraw(uint256 tokenId) external override {
        require(  _msgSender() == ownerOf(tokenId), "ChildMintableERC721: INVALID_TOKEN_OWNER" ); // _msgSender() 
        withdrawnTokens[tokenId] = true;
        _burn(tokenId);
    }

    function withdrawWithMetadata(uint256 tokenId) external override {
        require( msgSender() == ownerOf(tokenId), "ChildMintableERC721: INVALID_TOKEN_OWNER" );  // msgSender() 
        withdrawnTokens[tokenId] = true;
        // Encoding metadata associated with tokenId & emitting event
        emit TransferWithMetadata( ownerOf(tokenId), address(0), tokenId, this.encodeTokenMetadata(tokenId) );
        _burn(tokenId);
    }

RCNftHubL1.sol:      _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());
RCNftHubL2.sol:      _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());
RCTreasury.sol:        _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());
RCTreasury.sol:        _setupRole(UBER_OWNER,               _msgSender());
RCTreasury.sol:        _setupRole(OWNER,                         _msgSender());
RCTreasury.sol:        _setupRole(GOVERNOR,                   _msgSender());
RCTreasury.sol:        _setupRole(WHITELIST,                    _msgSender());

## Tools Used
grep

## Recommended Mitigation Steps
Doublecheck the use of  _msgSender() in withdraw and adjust if necessary.

Add comments when using  _msgSender() 

Consider overriding _msgSender(), as is done in the example below:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/metatx/ERC2771Context.sol



