## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [RCFactory: Solve stack too deep for getMarketInfo()](https://github.com/code-423n4/2021-08-realitycards-findings/issues/41) 

# Handle

hickuphh3


# Vulnerability details

### Impact

The `marketInfoResults` is a parameter used by `getMarketInfo()` to determine the length of results to return. As the `setMarketInfoResults()` comments state, "(it) would be better to pass this as a parameter in getMarketInfo.. however we are limited because of stack too deep errors".

This limitation can be overcome by defining the return array variables as the function output, as suggested below.

The need for `marketInfoResults` and its setter function is then made redundant, whilst making querying results of possibly varying lengths more convenient.

### Recommended Mitigation Steps

```jsx
function getMarketInfo(
  IRCMarket.Mode _mode,
  uint256 _state,
  uint256 _skipResults,
  uint256 _numResults // equivalent of marketInfoResults
)
  external
  view
  returns (
    address[] memory _marketAddresses,
    string[] memory _ipfsHashes,
    string[] memory _slugs,
    uint256[] memory _potSizes
	)
 {
	  uint256 _marketIndex = marketAddresses[_mode].length;
	  
	  _marketAddresses = new address[](_numResults);
	  _ipfsHashes = new string[](_numResults);
	  _slugs = new string[](_numResults);
	  _potSizes = new uint256[](_numResults);
		...
}
```

