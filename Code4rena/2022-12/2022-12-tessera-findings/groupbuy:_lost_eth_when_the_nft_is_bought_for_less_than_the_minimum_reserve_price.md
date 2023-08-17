## Tags

- bug
- 3 (High Risk)
- disagree with severity
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-02

# [GroupBuy: Lost ETH when the NFT is bought for less than the minimum reserve price](https://github.com/code-423n4/2022-12-tessera-findings/issues/7) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/1e408ebc1c4fdcc72678ea7f21a94d38855ccc0b/src/modules/GroupBuy.sol#L242


# Vulnerability details

## Impact
The `purchase` function does not require that an NFT is bought for exactly `minReservePrices[_poolId] * filledQuantities[_poolId]`, the price is only not allowed to be greater:
```solidity
if (_price > minReservePrices[_poolId] * filledQuantities[_poolId])
            revert InvalidPurchase();
```
This makes sense because it is not sensible to pay more when the purchase also succeeds with a smaller amount. However, the logic within `claim` does assume that the NFT was bought for `minReservePrices[_poolId]`. It decreases from `contribution` the quantity times the reserve price for all bids:
```solidity
contribution -= quantity * reservePrice;
```
Only the remaining amount is reimbursed to the user, which can lead to a loss of funds.

## Proof Of Concept
Let's say that `filledQuantities[_poolId] = 100` and `minReservePrices[_poolId]` (i.e., the lowest bid) was 1 ETH. However, it was possible to buy the NFT for only 50 ETH. When a user has contributed 20 * 1 ETH, he does not get anything back when calling `claim`, although only 10 ETH (0.5 ETH * 20) of his contributions were used to buy the NFT. The overall loss of funds for all contributors is 50 ETH.

## Recommended Mitigation Steps
Set `minReservePrices[_poolId]` to `_price / filledQuantities[_poolId]` after a purchase.