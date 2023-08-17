## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [ Someone can create non-liquidatable auction if the collateral asset fails on transferring to address(0)](https://github.com/code-423n4/2022-07-yield-findings/issues/116) 

# Lines of code

https://github.com/code-423n4/2022-07-yield/blob/main/contracts/Witch.sol#L176
https://github.com/code-423n4/2022-07-yield/blob/6ab092b8c10e4dabb470918ae15c6451c861655f/contracts/Witch.sol#L399


# Vulnerability details

## Impact
might lead to systematic debt. Cause errors for liquidators to run normally.

## Proof of Concept
In the function `auction`, there is on input validation around whether the `to` is `address(0)` or not. and if the `auctioneerReward` is set to an value > 0 (as default),  each liquidate call will call `Join` module to pay out to `auctioneer` with the following line:

```jsx
if (auctioneerCut > 0) {
    ilkJoin.exit(auction_.auctioneer, auctioneerCut.u128());
}
```

This line will revert if `auctioneer` is set to `address(0)` on some tokens (revert on transferring to address(0) is a [default behaviour of the OpenZeppelin template](https://www.notion.so/Yield-Witch-555e6981c26b41008d03a504077b4770)). So if someone start an `auction` with `to = address(0)`, this auction becomes un-liquidatable.

A malicious user can run a bot to monitor his own vault, and if the got underwater and they don’t have enough collateral to top up, they can immediately start an auction on their own vault and set actioneer to `0` to avoid actually being liquidated, which breaks the design of the system.


## Recommended Mitigation Steps

Add check while starting an auction:

```jsx
function auction(bytes12 vaultId, address to)
    external
    returns (DataTypes.Auction memory auction_)
{
    require (to != address(0), "invalid auctioneer");
		...
}		
```

