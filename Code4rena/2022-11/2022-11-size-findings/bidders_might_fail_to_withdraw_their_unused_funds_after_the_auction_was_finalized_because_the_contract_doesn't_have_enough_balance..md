## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-01

# [Bidders might fail to withdraw their unused funds after the auction was finalized because the contract doesn't have enough balance.](https://github.com/code-423n4/2022-11-size-findings/issues/94) 

# Lines of code

https://github.com/code-423n4/2022-11-size/blob/706a77e585d0852eae6ba0dca73dc73eb37f8fb6/src/SizeSealed.sol#L325


# Vulnerability details

## Impact
Bidders might fail to withdraw their unused funds after the auction was finalized because the contract doesn't have enough balance.

The main flaw is the seller might receive more quote tokens than the bidders offer after the auction was finalized.

If there is no other auctions to use the same quote token, the last bidder will fail to withdraw his funds because the contract doesn't have enough balance of quote token.

## Proof of Concept
After the auction was finalized, the seller receives the `filledQuote` amount of quote token using [data.filledBase](https://github.com/code-423n4/2022-11-size/blob/706a77e585d0852eae6ba0dca73dc73eb37f8fb6/src/SizeSealed.sol#L325).

```solidity
    // Calculate quote amount based on clearing price
    uint256 filledQuote = FixedPointMathLib.mulDivDown(clearingQuote, data.filledBase, clearingBase);
```

But when the bidders withdraw the funds using `withdraw()`, they offer the quote token [using this formula](https://github.com/code-423n4/2022-11-size/blob/706a77e585d0852eae6ba0dca73dc73eb37f8fb6/src/SizeSealed.sol#L375-L382).

```solidity
    // Refund unfilled quoteAmount on first withdraw
    if (b.quoteAmount != 0) {
        uint256 quoteBought = FixedPointMathLib.mulDivDown(baseAmount, a.data.lowestQuote, a.data.lowestBase);
        uint256 refundedQuote = b.quoteAmount - quoteBought;
        b.quoteAmount = 0;

        SafeTransferLib.safeTransfer(ERC20(a.params.quoteToken), msg.sender, refundedQuote);
    }
```

Even if they use the same clearing price, the total amount of quote token that the bidders offer might be less than the amount that the seller charged during finalization because the round down would happen several times with the bidders.

This is the test to show the scenario.

```solidity
    function testAuditBidderMoneyLock() public {
        // in this scenario, we show that bidder's money can be locked due to inaccurate calculation of claimed quote tokens for a seller
        uint128 K = 1 ether;
        baseToSell = 4*K;
        uint256 aid = seller.createAuction(
            baseToSell, reserveQuotePerBase, minimumBidQuote, startTime, endTime, unlockTime, unlockEnd, cliffPercent
        );

        bidder1.setAuctionId(aid);
        bidder1.bidOnAuctionWithSalt(3*K, 3*K+2, "Honest bidder");
        bidder2.setAuctionId(aid);
        bidder2.bidOnAuctionWithSalt(2*K, 2*K+1, "Honest bidder");

        vm.warp(endTime);

        uint256[] memory bidIndices = new uint[](2);
        bidIndices[0] = 0;
        bidIndices[1] = 1;

        seller.finalize(bidIndices, 2*K, 2*K+1);
        emit log_string("Seller claimed");
        // seller claimed 4*K+2
        assertEq(quoteToken.balanceOf(address(seller)), 4*K+2);
        // contract has K+1 quote token left
        assertEq(quoteToken.balanceOf(address(auction)), K+1);

        // bidder1 withdraws
        bidder1.withdraw();
        emit log_string("Bidder 1 withdrew");
        // contract has K quote token left
        assertEq(quoteToken.balanceOf(address(auction)), K);
        // bidder2 withdraws and he is supposed to be able to claim K+1 quote tokens
        // but the protocol reverts because of insufficient quote tokens
        bidder2.withdraw();
        emit log_string("Bidder 2 withdrew"); // will not happen
    }
```

The test result shows the seller charged more quote token than the bidders offer so the last bidder can't withdraw his unused quote token because the contract doesn't have enough balance.

```solidity
    Running 1 test for src/test/SizeSealed.t.sol:SizeSealedTest
    [FAIL. Reason: TRANSFER_FAILED] testAuditBidderMoneyLock() (gas: 954985)
    Logs:
    Seller claimed
    Bidder 1 withdrew

    Test result: FAILED. 0 passed; 1 failed; finished in 6.94ms

    Failing tests:
    Encountered 1 failing test in src/test/SizeSealed.t.sol:SizeSealedTest
    [FAIL. Reason: TRANSFER_FAILED] testAuditBidderMoneyLock() (gas: 954985)
```

## Tools Used
Foundry

## Recommended Mitigation Steps
Currently, the `FinalizeData` struct contains the `filledBase` only and calculates the `filledQuote` using the clearing price.

```solidity
    struct FinalizeData {
        uint256 reserveQuotePerBase;
        uint128 totalBaseAmount;
        uint128 filledBase;
        uint256 previousQuotePerBase;
        uint256 previousIndex;
    }
```

I think we should add one more field `filledQuote` and update it during auction finalization.

And the seller can recieve the sum of `filledQuote` of all bidders to avoid the rounding issue.

Also, each bidder can pay the `filledQuote` of quote token and receive the `filledBase` of base token without calculating again using the clearing price.