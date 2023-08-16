## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Refactor `FeeSplitter::getAmountDue` to save one variable slot](https://github.com/code-423n4/2021-11-nested-findings/issues/68) 

# Handle

pmerkleplant


# Vulnerability details

## Impact
Function `getAmountDue` in `FeeSplitter.sol` defines the variable 
`totalReceived` in [line 83](https://github.com/code-423n4/2021-11-nested/blob/main/contracts/FeeSplitter.sol#L83) eventhough it is already known if the variable is
even necessary.

The variable is uneccessary if `_tokenRecords.totalShares == 0`.
Not declaring it, if not necessary, saves gas.

## Recommended Mitigation Steps
Rewrite the function to something like:
```
TokenRecords storage _tokenRecords = tokenRecords[address(_token)];
if (_tokenRecords.totalShares == 0) return 0;
uint256 totalReceived = _tokenRecords.totalReleased + _token.balanceOf(address(this));
// Rest same as before
```

