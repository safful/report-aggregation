## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-08

# [ Earlier bidders get cut out of future NFT holdings by bidders specifying the same price.](https://github.com/code-423n4/2022-12-tessera-findings/issues/45) 

# Lines of code

LOC: https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L301


# Vulnerability details

## Description

In GroupBuy module, users can call contribute to get a piece of the NFT pie. There are two stages in transforming the msg.value to holdings in the NFT. 

1. filling at any price(supply is not yet saturated)
```
uint256 fillAtAnyPriceQuantity = remainingSupply < _quantity ? remainingSupply : _quantity;
// Checks if quantity amount being filled is greater than 0
if (fillAtAnyPriceQuantity > 0) {
    // Inserts bid into end of queue
    bidPriorityQueues[_poolId].insert(msg.sender, _price, fillAtAnyPriceQuantity);
    // Increments total amount of filled quantities
    filledQuantities[_poolId] += fillAtAnyPriceQuantity;
}
```

2. Trim out lower price offers to make room for current higher offer.
```
// Calculates unfilled quantity amount based on desired quantity and actual filled quantity amount
uint256 unfilledQuantity = _quantity - fillAtAnyPriceQuantity;
// Processes bids in queue to recalculate unfilled quantity amount
unfilledQuantity = processBidsInQueue(_poolId, unfilledQuantity, _price);
```

The while loop in `processBidsInQueue` will keep removing existing bids with lower price and create new queue entries for currently processed bid. When it reached a bid with a higher price than msg.sender's price, it will break:
```
while (quantity > 0) {
    // Retrieves lowest bid in queue
    Bid storage lowestBid = bidPriorityQueues[_poolId].getMin();
    // Breaks out of while loop if given price is less than than lowest bid price
    if (_price < lowestBid.price) {
        break;
    }
```

The issue is that when `_price  == lowestBid.price`, we don't break and current bid will kick out older bid, as can be seen here:

```
// Decrements given quantity amount from lowest bid quantity
lowestBid.quantity -= quantity;
// Calculates partial contribution of bid by quantity amount and price
uint256 contribution = quantity * lowestBid.price;
// Decrements partial contribution amount of lowest bid from total and user contributions
totalContributions[_poolId] -= contribution;
userContributions[_poolId][lowestBid.owner] -= contribution;
// Increments pending balance of lowest bid owner
pendingBalances[lowestBid.owner] += contribution;
// Inserts new bid with given quantity amount into proper position of queue
bidPriorityQueues[_poolId].insert(msg.sender, _price, quantity);
```

The described behavior goes against what the docs [describe](https://github.com/code-423n4/2022-12-tessera#step-3-other-users-deposit-funds-to-pool-filtering) will happen when two equal priced bids collide.

## Impact

Earlier bidders get cut out of future NFT holdings by bidders specifying the same price.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Change the < to <= in the if condition:
```
if (_price <= lowestBid.price) {
    break;
}
```