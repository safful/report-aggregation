## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [NestedFactory: in deleteAsset and freeToken, tokens should only be declared once ](https://github.com/code-423n4/2021-11-nested-findings/issues/49) 

# Handle

PierrickGT


# Vulnerability details

## Impact
In [deleteAsset](https://github.com/code-423n4/2021-11-nested/blob/cbd39fe7d76ed8c84eb767a5f3b6eba83e034656/contracts/NestedRecords.sol#L213), `tokens` is declared once in the function and then a second time in the function `freeToken`.

## Proof of Concept
The `freeToken` function being used only in `deleteAsset`, the code from this function can be moved to `deleteAsset` and the function removed. This way, we don't have to pass `tokens` to the `freeToken` function and we avoid declaring it here a second time.

## Recommended Mitigation Steps
The following change is recommended.

```
function deleteAsset(uint256 _nftId, uint256 _tokenIndex) public onlyFactory {
    address[] storage tokens = records[_nftId].tokens;
    address token = tokens[_tokenIndex];
    Holding memory holding = records[_nftId].holdings[token];

    require(holding.isActive, "NestedRecords: HOLDING_INACTIVE");

    delete records[_nftId].holdings[token];
    tokens[_tokenIndex] = tokens[tokens.length - 1];
    tokens.pop();
}
```

