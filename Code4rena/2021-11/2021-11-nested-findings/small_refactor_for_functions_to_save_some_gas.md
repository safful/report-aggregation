## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Small refactor for functions to save some gas](https://github.com/code-423n4/2021-11-nested-findings/issues/193) 

# Handle

0xngndev


# Vulnerability details

## Impact
In `FeeSplitter.sol` by doing a small refactory gas can be saved in case of a revert in the functions: `getAmountDue` and `_releaseToken` . We can swap the order of two lines so we return earlier in case of a bad input, this way we save some gas because the evm would execute less opcodes before reverting.

## Mitigation steps
getAmountDue: Swap line 83 with 84 to avoid computing unnecessary logic. Remove the "else"  and combine it with line 83. Something like this:

```
  function getAmountDue(address _account, IERC20 _token) public view returns (uint256) {
        TokenRecords storage _tokenRecords = tokenRecords[address(_token)];
				if (_tokenRecords.totalShares == 0) return 0;
        uint256 totalReceived = _tokenRecords.totalReleased + _token.balanceOf(address(this));
        uint256 amountDue = (totalReceived * _tokenRecords.shares[_account]) /
            _tokenRecords.totalShares -
            _tokenRecords.released[_account];
        return amountDue;
    }
```
_releaseToken: move line 252 after the require in line 254. Like this:

```
  function _releaseToken(address _account, IERC20 _token) private returns (uint256) {
        uint256 amountToRelease = getAmountDue(_account, _token);
        require(amountToRelease != 0, "FeeSplitter: NO_PAYMENT_DUE");
        TokenRecords storage _tokenRecords = tokenRecords[address(_token)];

        _tokenRecords.released[_account] = _tokenRecords.released[_account] + amountToRelease;
        _tokenRecords.totalReleased = _tokenRecords.totalReleased + amountToRelease;

        return amountToRelease;
    }
```


