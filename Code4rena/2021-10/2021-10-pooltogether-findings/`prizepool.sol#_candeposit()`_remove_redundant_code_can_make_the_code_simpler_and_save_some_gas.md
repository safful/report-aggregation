## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- resolved

# [`PrizePool.sol#_canDeposit()` Remove redundant code can make the code simpler and save some gas](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/37) 

# Handle

WatchPug


# Vulnerability details

https://github.com/pooltogether/v4-core/blob/055335bf9b09e3f4bbe11a788710dd04d827bf37/contracts/prize-pool/PrizePool.sol#L361-L368

```solidity
function _canDeposit(address _user, uint256 _amount) internal view returns (bool) {
    ITicket _ticket = ticket;
    uint256 _balanceCap = balanceCap;

    if (_balanceCap == type(uint256).max) return true;

    return (_ticket.balanceOf(_user) + _amount <= _balanceCap);
}
```

`ITicket _ticket = ticket;` is redundant, removing it will also avoid a `sload` if returned when `_balanceCap == type(uint256).max`.

### Recommendation

Change to:

```solidity
function _canDeposit(address _user, uint256 _amount) internal view returns (bool) {

    uint256 _balanceCap = balanceCap;

    if (_balanceCap == type(uint256).max) return true;

    return (ticket.balanceOf(_user) + _amount <= _balanceCap);
}
```

