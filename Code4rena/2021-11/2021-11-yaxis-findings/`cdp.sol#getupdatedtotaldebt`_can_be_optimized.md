## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`CDP.sol#getUpdatedTotalDebt` can be optimized](https://github.com/code-423n4/2021-11-yaxis-findings/issues/92) 

# Handle

0x0x0x


# Vulnerability details

## Proof of Concept
Current implementation has two if statements, but actually the same logic can be coded with only one if statement. Since `_unclaimedYield == 0` is a special case of `_unclaimedYield < _currentTotalDebt` and does not require any extra code.

```
  function getUpdatedTotalDebt(Data storage _self, Context storage _ctx) internal view returns (uint256) {
    uint256 _unclaimedYield = _self.getEarnedYield(_ctx);
    uint256 _currentTotalDebt = _self.totalDebt;

    if (_unclaimedYield < _currentTotalDebt) {
      return _currentTotalDebt - _unclaimedYield;
    }
    else {
      return 0;
    }
  }
```
## Tools Used
Manual analysis

