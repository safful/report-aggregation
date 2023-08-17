## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-03

# [The sorting logic is not strict enough](https://github.com/code-423n4/2022-11-size-findings/issues/97) 

# Lines of code

https://github.com/code-423n4/2022-11-size/blob/706a77e585d0852eae6ba0dca73dc73eb37f8fb6/src/SizeSealed.sol#L269-L277


# Vulnerability details

## Impact
When the seller finalizes his auction, all bids are sorted according to the `quotePerBase` and it's calculated using the `FixedPointMathLib.mulDivDown()`.

And the earliest bid will be used first for the same `quotePerBase` but this ratio is not strict enough so that the worse bid might be filled than the better one.

As a result, the seller might receive fewer quote token than he wants.

## Proof of Concept
This is the test to show the scenario.

```solidity
    function testAuditWrongSorting() public {
        // this test will show that it is possible the seller can not claim the best bid because of the inaccurate comparison in finalization
        uint128 K = 1<<64;
        baseToSell = K + 2;
        uint256 aid = seller.createAuction(
            baseToSell, reserveQuotePerBase, minimumBidQuote, startTime, endTime, unlockTime, unlockEnd, cliffPercent
        );

        bidder1.setAuctionId(aid);
        bidder1.bidOnAuctionWithSalt(K+1, K, "Worse bidder");
        bidder2.setAuctionId(aid);
        bidder2.bidOnAuctionWithSalt(K+2, K+1, "Better bidder"); // This is the better bid because (K+1)/(K+2) > K/(K+1)

        vm.warp(endTime);

        uint256[] memory bidIndices = new uint[](2);
        bidIndices[0] = 1; // the seller is smart enough to choose the correct order to (1, 0)
        bidIndices[1] = 0;

        vm.expectRevert(ISizeSealed.InvalidSorting.selector);
        seller.finalize(bidIndices, K+2, K+1); // this reverts because of #273

        // next the seller is forced to call the finalize with parameter K+1, K preferring the first bidder
        bidIndices[0] = 0;
        bidIndices[1] = 1;
        seller.finalize(bidIndices, K+1, K);

        // at this point the seller gets K quote tokens while he could get K+1 quote tokens with the better bidder
        assertEq(quoteToken.balanceOf(address(seller)), K);
    }
```

This is the output of the test.

```solidity
    Running 1 test for src/test/SizeSealed.t.sol:SizeSealedTest
    [PASS] testAuditWrongSorting() (gas: 984991)
    Test result: ok. 1 passed; 0 failed; finished in 7.22ms
```

When it calculates the [quotePerBase](https://github.com/code-423n4/2022-11-size/blob/706a77e585d0852eae6ba0dca73dc73eb37f8fb6/src/SizeSealed.sol#L269), they are same each other with `(base, quote) = (K+1, K) and (K+2, K+1) when K = 1<<64`.

So the seller can receive `K+1` of the quote token but he got `K`.

I think `K` is realistic enough with the 18 decimals token because K is around 18 * 1e18.

## Tools Used
Foundry

## Recommended Mitigation Steps
As we can see from the test, it's not strict enough to compare bidders using `quotePerBase`.

We can compare them by multiplying them like below.

$\frac {quote1}{base1} >= \frac{quote2}{base2} <=> quote1 * base2 >= quote2 * base1 $

So we can add 2 elements to `FinalizeData` struct and modify [this comparison](https://github.com/code-423n4/2022-11-size/blob/706a77e585d0852eae6ba0dca73dc73eb37f8fb6/src/SizeSealed.sol#L269-L277) like below.

```solidity
    struct FinalizeData {
        uint256 reserveQuotePerBase;
        uint128 totalBaseAmount;
        uint128 filledBase;
        uint256 previousQuotePerBase;
        uint256 previousIndex;
        uint128 previousQuote; //++++++++++++
        uint128 previousBase; //+++++++++++
    }
```

```solidity
    uint256 quotePerBase = FixedPointMathLib.mulDivDown(b.quoteAmount, type(uint128).max, baseAmount);
    if (quotePerBase >= data.previousQuotePerBase) {
        // If last bid was the same price, make sure we filled the earliest bid first
        if (quotePerBase == data.previousQuotePerBase) {
            uint256 currentMult = uint256(b.quoteAmount) * data.previousBase; //mult for current bid
            uint256 previousMult = uint256(data.previousQuote) * baseAmount; //mult for the previous bid

            if (currentMult > previousMult) { // current bid is better
                revert InvalidSorting();    
            }

            if (currentMult == previousMult && data.previousIndex > bidIndex) revert InvalidSorting();
        } else {
            revert InvalidSorting();
        }
    }

    ...

    data.previousBase = baseAmount;
    data.previousQuote = b.quoteAmount;
```