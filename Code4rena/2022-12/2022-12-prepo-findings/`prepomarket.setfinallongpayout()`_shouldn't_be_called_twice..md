## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-05

# [`PrePOMarket.setFinalLongPayout()` shouldn't be called twice.](https://github.com/code-423n4/2022-12-prepo-findings/issues/231) 

# Lines of code

https://github.com/prepo-io/prepo-monorepo/blob/3541bc704ab185a969f300e96e2f744a572a3640/apps/smart-contracts/core/contracts/PrePOMarket.sol#L119


# Vulnerability details

## Impact
If `finalLongPayout` is changed twice by admin fault, the market would be insolvent as it should pay more collateral than it has.

## Proof of Concept
If `finalLongPayout` is less than `MAX_PAYOUT`, it means the market is ended and `longToken Price = finalLongPayout, shortToken Price = MAX_PAYOUT - finalLongPayout`.

So when users redeem their long/short tokens, the total amount of collateral tokens will be the same as the amount that users transferred during mint().

Btw in `setFinalLongPayout()`, there is no validation that this function can't be called twice and the below scenario would be possible.

1. Let's assume there is one user `Bob` in the market for simplicity.
2. `Bob` transferred 100 amounts of `collateral` and got 100 long/short tokens. The market has 100 `collateral`.
3. The market admin set `finalLongPayout = 60 * 1e16` and `Bob` redeemed 100 `longToken` and received 60 `collateral`. The market has 40 `collateral` now.
4. After that, the admin realized `finalLongPayout` is too high and changed `finalLongPayout = 40 * 1e16` again.
5. `Bob` tries to redeem 100 `shortToken` and receive 60 `collateral` but the market can't offer as it has 40 `collateral` only.

When there are several users in the market, some users can't redeem their long/short tokens as the market doesn't have enough `collaterals`.

## Tools Used
Manual Review

## Recommended Mitigation Steps
We should modify `setFinalLongPayout()` like below so it can't be finalized twice.

```solidity
  function setFinalLongPayout(uint256 _finalLongPayout) external override onlyOwner { 
    require(finalLongPayout > MAX_PAYOUT, "Finalized already"); //++++++++++++++++++++++++

    require(_finalLongPayout >= floorLongPayout, "Payout cannot be below floor");
    require(_finalLongPayout <= ceilingLongPayout, "Payout cannot exceed ceiling");
    finalLongPayout = _finalLongPayout;
    emit FinalLongPayoutSet(_finalLongPayout);
  }
```