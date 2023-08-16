## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Typos](https://github.com/code-423n4/2021-11-unlock-findings/issues/205) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinLockMetadata.sol#L109-L140

```solidity=109
  function tokenURI(
    uint256 _tokenId
  ) external
    view
    returns(string memory)
  {
    string memory URI;
    string memory tokenId;
    string memory lockAddress = address(this).address2Str();
    string memory seperator;

    if(_tokenId != 0) {
      tokenId = _tokenId.uint2Str();
    } else {
      tokenId = '';
    }

    if(bytes(baseTokenURI).length == 0) {
      URI = unlockProtocol.globalBaseTokenURI();
      seperator = '/';
    } else {
      URI = baseTokenURI;
      seperator = '';
      lockAddress = '';
    }

    return URI.strConcat(
        lockAddress,
        seperator,
        tokenId
      );
  }
```

`seperator` should be `separator`.

https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinRefunds.sol#L40-L43

```solidity=40
/**
   * @dev Invoked by the lock owner to destroy the user's ket and perform a refund and cancellation
   * of the key
   */
```

`user's ket` should be `user's key`.

