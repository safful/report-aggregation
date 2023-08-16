## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`updateShareholder` in `FeeSplitter.sol` can be implemented more efficiently](https://github.com/code-423n4/2021-11-nested-findings/issues/11) 

# Handle

0x0x0x


# Vulnerability details

## Explanation

`updateShareholder` in `FeeSplitter.sol` can be implemented more efficiently. The updated version consumes less gas and also has the second `require` statement earlier, which reduces  the gas cost in case the statement of second `require` is not fullfilled. 

`FeeSplitter.sol` : L166-174:
```
    function updateShareholder(uint256 _accountIndex, uint256 _weight) external onlyOwner {
        require(_accountIndex + 1 <= shareholders.length, "FeeSplitter: INVALID_ACCOUNT_INDEX");
        uint256 _totalWeights = totalWeights;
        _totalWeights -= shareholders[_accountIndex].weight;
        shareholders[_accountIndex].weight = _weight;
        _totalWeights += _weight;
        require(_totalWeights > 0, "FeeSplitter: TOTAL_WEIGHTS_ZERO");
        totalWeights = _totalWeights;
    }
```
can be replaced with:
```
    function updateShareholder(uint256 _accountIndex, uint256 _weight) external onlyOwner {
        require(_accountIndex + 1 <= shareholders.length, "FeeSplitter: INVALID_ACCOUNT_INDEX");
        totalWeights = totalWeights + _weight - shareholders[_accountIndex].weight;
        require(_totalWeights > 0, "FeeSplitter: TOTAL_WEIGHTS_ZERO");
        shareholders[_accountIndex].weight = _weight;
    }
```
## Tools Used

Manual analysis

