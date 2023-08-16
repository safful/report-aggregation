## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`CDP.sol#update.sol` can be optimized](https://github.com/code-423n4/2021-11-yaxis-findings/issues/91) 

# Handle

0x0x0x


# Vulnerability details

## Concept
The current code is:
```
  function update(Data storage _self, Context storage _ctx) internal {
    uint256 _earnedYield = _self.getEarnedYield(_ctx);
    if (_earnedYield > _self.totalDebt) {
      uint256 _currentTotalDebt = _self.totalDebt;
      _self.totalDebt = 0;
      _self.totalCredit = _earnedYield.sub(_currentTotalDebt);
    } else {
      _self.totalDebt = _self.totalDebt.sub(_earnedYield);
    }
    _self.lastAccumulatedYieldWeight = _ctx.accumulatedYieldWeight;
  }
```
We cache _self.totalDebt, but it is not required, since we can use it before we change it. This code block can be replaced with:
```
  function update(Data storage _self, Context storage _ctx) internal {
    uint256 _earnedYield = _self.getEarnedYield(_ctx);
    if (_earnedYield > _self.totalDebt) {
      _self.totalCredit = _earnedYield.sub(_self.totalDebt);
      _self.totalDebt = 0;
    } else {
      _self.totalDebt = _self.totalDebt.sub(_earnedYield);
    }
    _self.lastAccumulatedYieldWeight = _ctx.accumulatedYieldWeight;
  }
```
By doing so, we don't cache `_self.totalDebt` just to use it once.

