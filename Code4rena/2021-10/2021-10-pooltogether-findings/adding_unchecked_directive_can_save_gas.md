## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- resolved

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/38) 

# Handle

WatchPug


# Vulnerability details

## Adding unchecked directive can save gas

For the arithmetic operations that will never over/underflow, using the unchecked directive (Solidity v0.8 has default overflow/underflow checks) can save some gas from the unnecessary internal over/underflow checks.

For example:

1. `PrizePool.sol#award()`

https://github.com/pooltogether/v4-core/blob/055335bf9b09e3f4bbe11a788710dd04d827bf37/contracts/prize-pool/PrizePool.sol#L210-L225

```solidity
function award(address _to, uint256 _amount) external override onlyPrizeStrategy {
    if (_amount == 0) {
        return;
    }

    uint256 currentAwardBalance = _currentAwardBalance;

    require(_amount <= currentAwardBalance, "PrizePool/award-exceeds-avail");
    _currentAwardBalance = currentAwardBalance - _amount;

    ITicket _ticket = ticket;

    _mint(_to, _amount, _ticket);

    emit Awarded(_to, _ticket, _amount);
}
```

`currentAwardBalance - _amount;` will never underflow.


2. `PrizeDistributor.sol#claim()`

https://github.com/pooltogether/v4-core/blob/055335bf9b09e3f4bbe11a788710dd04d827bf37/contracts/PrizeDistributor.sol#L74-L77

```solidity
if (payout > oldPayout) {
    payoutDiff = payout - oldPayout;
    _setDrawPayoutBalanceOf(_user, drawId, payout);
}
```

`payout - oldPayout` will never underflow.

