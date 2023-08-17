## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-09

# [GroupBuys that are completely filled still don't raise stated target amount](https://github.com/code-423n4/2022-12-tessera-findings/issues/49) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L83


# Vulnerability details

## Description

createPool() in GroupBuy.sol creates a new contribution pool around an NFT. It specifies a target \_initialPrice as minimum amount of ETH the NFT will cost, and \_totalSupply which is the number of Raes to be minted on purchase success.

minBidPrices is calculated from the two numbers. All future bids must be at least minBidPrices. It is assumed that if the totalSupply of Raes is filled up, the group will collect the initialPrice.
```
// Calculates minimum bid price based on initial price of NFT and desired total supply
minBidPrices[currentId] = _initialPrice / _totalSupply;
```

The issue is that division rounding error will make minBidPrices too low. Therefore, when all Raes are minted using minBidPrices price:
minBidPrices[currentId] * \_totalSupply != \_initialPrice 

Therefore, not enough money has been collected to fund the purchase.
It can be assumed that most people will use minBidPrices to drive the price they will choose. Therefore, even after discovering that the Group has not raised enough after filling the supply pool, it will be very hard to get everyone to top up the contribution by a bit. This is because the settled price which is collected from all contributions is minReservePrices, which is always the minimum price deposited.

Code in contribute that updates minReservePrices:
```
// Updates minimum reserve price if filled quantity amount is greater than 0
if (filledQuantity > 0) minReservePrices[_poolId] = getMinPrice(_poolId);
```


The check in purchase() that we don't charge more than minReservePrices from each contribution:
```
if (_price > minReservePrices[_poolId] * filledQuantities[_poolId])
    revert InvalidPurchase();
```

We can see an important contract functionality is not working as expected which will impair NFT purchases.

## Impact

GroupBuys that are completely filled still don't raise stated target amount

## Tools Used

Manual audit

## Recommended Mitigation Steps

Round the minBidPrices up, rather than down. It will ensure enough funds are collected.