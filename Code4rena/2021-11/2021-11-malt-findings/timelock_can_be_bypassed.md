## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Timelock can be bypassed](https://github.com/code-423n4/2021-11-malt-findings/issues/263) 

# Handle

WatchPug


# Vulnerability details

The purpose of a Timelock contract is to put a limit on the privileges of the `governor`, by forcing a two step process with a preset delay time.

However, we found that the current implementation actually won't serve that purpose as it allows the `governor` to execute any transactions without any constraints.

To do that, the current governor can call `Timelock#setGovernor(address _governor)` and set a new `governor` effective immediately.

And the new `governor` can then call `Timelock#setDelay()` and change the delay to `0`, also effective immediately.

The new `governor` can now use all the privileges without a delay, including granting minter role to any address and mint unlimited amount of MALT.

In conclusion, a Timelock contract is supposed to guard the protocol from lost private key or malicious actions. The current implementation won't fulfill that mission.

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Timelock.sol#L98-L105
```solidity=98{100,102-103}
  function setGovernor(address _governor)
    public
    onlyRole(GOVERNOR_ROLE, "Must have timelock role")
  {
    _swapRole(_governor, governor, GOVERNOR_ROLE);
    governor = _governor;
    emit NewGovernor(_governor);
  }
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Timelock.sol#L66-L77
```solidity=66{71,74}
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


## Recommendation

Consider making `setGovernor` and `setDelay` only callable from the Timelock contract itself.

Specificaly, changing from `onlyRole(GOVERNOR_ROLE, "Must have timelock role")` to `require(msg.sender == address(this), "...")`.

Also, consider changing `_adminSetup(_admin)` in `Timelock#initialize()` to `_adminSetup(address(this))`, so that all roles are managed by the timelock itself as well.

