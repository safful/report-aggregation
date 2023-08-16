## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [tokenByIndex returns wrong token id](https://github.com/code-423n4/2021-11-unlock-findings/issues/222) 

# Handle

pauliax


# Vulnerability details

## Impact
function _assignNewTokenId first increments _totalSupply and then assigns token id, so ids start from 1, not 0. However, function tokenByIndex in MixinERC721Enumerable expects the index to be less than totalSupply:
```solidity
  /// @notice Enumerate valid NFTs
  /// @dev Throws if `_index` >= `totalSupply()`.
  /// @param _index A counter less than `totalSupply()`
  /// @return The token identifier for the `_index`th NFT,
  ///  (sort order not specified)
  function tokenByIndex(
    uint256 _index
  ) public view
    returns (uint256)
  {
    require(_index < _totalSupply, 'OUT_OF_RANGE');
    return _index;
  }
```
This mismatch between indexes and token ids may trick other platforms or integrations.

## Recommended Mitigation Steps
I think the solution is simply returning index + 1:
```solidity
    require(_index < _totalSupply, 'OUT_OF_RANGE');
    return _index + 1; // index 0 = token id 1
```

