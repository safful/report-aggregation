## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Checking `uint256` variables `>= 0` is redundant](https://github.com/code-423n4/2021-11-malt-findings/issues/309) 

# Handle

WatchPug


# Vulnerability details

Checking `uint256` variables >= 0 is redundant as they always >= 0.

Instances include:

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/SwingTrader.sol#L169-L172

```solidity
function setLpProfitCut(uint256 _profitCut) public onlyRole(ADMIN_ROLE, "Must have admin privs") {
    require(_profitCut >= 0 && _profitCut <= 1000, "Must be between 0 and 100%");
    lpProfitCut = _profitCut;  
  }
```

`_profitCut >= 0` at L170.

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Timelock.sol#L66-L77

```solidity
function setDelay(uint256 _delay)
    public
    onlyRole(GOVERNOR_ROLE, "Must have timelock role")
  {
    require(
      _delay >= 0 && _delay < gracePeriod,
      "Timelock::setDelay: Delay must not be greater equal to zero and less than gracePeriod"
    );
    delay = _delay;

    emit NewDelay(delay);
  }
```

`_delay >= 0` at L71.

