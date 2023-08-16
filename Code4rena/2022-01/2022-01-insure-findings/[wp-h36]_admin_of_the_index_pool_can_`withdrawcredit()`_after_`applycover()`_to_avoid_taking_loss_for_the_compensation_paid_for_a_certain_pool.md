## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[WP-H36] Admin of the index pool can `withdrawCredit()` after `applyCover()` to avoid taking loss for the compensation paid for a certain pool](https://github.com/code-423n4/2022-01-insure-findings/issues/281) 

# Handle

WatchPug


# Vulnerability details

In the current implementation, when an incident is reported for a certain pool, the index pool can still `withdrawCredit()` from the pool, which in the best interest of an index pool, the admin of the index pool is preferred to do so.

This allows the index pool to escape from the responsibility for the risks of invested pools.

Making the LPs of the pool take an unfair share of the responsibility.

### PoC

- Pool A `totalCredit` = 10,000
- Pool A `rewardPerCredit` = 1

1. [Index Pool 1] allocates 1,000 credits to Pool `A`:

- `totalCredit` = 11,000
- indicies[Index Pool 1] = 1,000

2. After a while, Pool A `rewardPerCredit` has grown to `1.1`, and `applyCover()` has been called, [Index Pool 1] call `withdrawCredit()` get 100 premium

- `totalCredit` = 10,000
- indicies[Index Pool 1] = 0

3. After `pendingEnd`, the pool `resume()`,[ Index Pool 1] will not be paying for the compensation since `credit` is 0.

In our case, [Index Pool 1] earned premium without paying for a part of the compensation.

### Recommendation

Change to:

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/PoolTemplate.sol#L416-L421

```solidity
    function withdrawCredit(uint256 _credit)
        external
        override
        returns (uint256 _pending)
    {
        require(
            marketStatus == MarketStatus.Trading,
            "ERROR: WITHDRAW_CREDIT_BAD_CONDITIONS"
        );
        IndexInfo storage _index = indicies[msg.sender];
```

