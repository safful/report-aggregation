## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Misleading error message](https://github.com/code-423n4/2021-11-nested-findings/issues/161) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-nested/blob/f646002b692ca5fa3631acfff87dda897541cf41/contracts/NestedRecords.sol#L56-L79

```solidity=56{67}
/// @notice Add a record for NFT data into our mappings
/// @param _nftId The id of the NFT
/// @param _token The address of the token
/// @param _amount The amount of tokens bought
/// @param _reserve The address of the reserve
function createRecord(
    uint256 _nftId,
    address _token,
    uint256 _amount,
    address _reserve
) public onlyFactory {
    require(records[_nftId].tokens.length < maxHoldingsCount, "NestedRecords: TOO_MANY_ORDERS");
    require(
        _reserve != address(0) && (_reserve == records[_nftId].reserve || records[_nftId].reserve == address(0)),
        "NestedRecords: INVALID_RESERVE"
    );

    Holding memory holding = records[_nftId].holdings[_token];
    require(!holding.isActive, "NestedRecords: HOLDING_EXISTS");

    records[_nftId].holdings[_token] = Holding({ token: _token, amount: _amount, isActive: true });
    records[_nftId].tokens.push(_token);
    records[_nftId].reserve = _reserve;
}
```

The error message "NestedRecords: TOO_MANY_ORDERS" should be changed to "NestedRecords: TOO_MANY_TOKENS".

