## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [NestedRecords: createRecord() can be made internal](https://github.com/code-423n4/2021-11-nested-findings/issues/124) 

# Handle

GreyArt


# Vulnerability details

## Impact

`createRecord()` is only invoked by `store()`. Its visibility can therefore be made internal / private.

## Recommended Mitigation Steps

```jsx
function createRecord(
  uint256 _nftId,
  address _token,
  uint256 _amount,
	address _reserve
) internal onlyFactory {...}
```

