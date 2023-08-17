## Tags

- bug
- 2 (Med Risk)
- selected for report
- sponsor confirmed
- M-10

# [BondNFT#extendLock force a user to extend the bond at least for current bond.period](https://github.com/code-423n4/2022-12-tigris-findings/issues/359) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/BondNFT.sol#L97-L125


# Vulnerability details

## Description
The current implementation forces a user to extend their bonds for at least they current bond period. These mean that, for instance, a bond which was initially locked for 365 can never be extended, even after a week of being created.

If we consider that a bond should have at least a 7 days lock and at the most 365 days, then the current ```BondNFT.extendLock``` function should be refactored.

## Impact
* Current ```BondNFT.extendLock``` function does not work as expected, forcing user who want to extend their bond to extend them at least for their current bond.period.
* For bonds which were set with a lock period of 365 days, they can not be extended, even after days of their creation.

## POC
```typescript
// In 09.Bond.js,  describe "Extending lock"
it("POC: Extending the lock does not work as expected", async function () {
      await stabletoken.connect(owner).mintFor(user.address, ethers.utils.parseEther("100"));
      // user lock bond funds for 10 days
      await lock.connect(user).lock(StableToken.address, ethers.utils.parseEther("100"), 10);

      const fiveDaysTime = 5 * 24 * 60 * 60
      const eightDaysTime = 8 * 24 * 60 * 60

      // owner distribute rewards
      console.log("User created a lock for 10 days")
      await stabletoken.connect(owner).mintFor(owner.address, ethers.utils.parseEther("10"));
      await bond.connect(owner).distribute(stabletoken.address, ethers.utils.parseEther("10"));

      // Five days pass
      await network.provider.send("evm_increaseTime", [fiveDaysTime]); // Skip 10 days
      await network.provider.send("evm_mine");
      console.log("\n5 days pass")

      // User decide to extend their lock three days, given the current implementation the user is forced to extended 13 days
      const bondInfoBeforeExtension = await bond.idToBond(1)
      console.log(`Bond info before extension: {period: ${bondInfoBeforeExtension.period}, expireEpoch: ${bondInfoBeforeExtension.expireEpoch}}`)
      
      await lock.connect(user).extendLock(1, 0, 3)
      console.log("Bond was extended for 3 days")
      const bondInfoAfterExtension = await bond.idToBond(1)
      console.log(`Bond info after extension: {period: ${bondInfoAfterExtension.period}, expireEpoch: ${bondInfoAfterExtension.expireEpoch}}`)

      // 8 days pass, user should be able to release the bond given the extension of 3 days (8 days should be enough)
      await network.provider.send("evm_increaseTime", [eightDaysTime]);
      await network.provider.send("evm_mine");
      console.log("\n8 days later")
      console.log("After 13 days (10 original days + 3 days from extension) the user can not release the bond")
      
      // The user decide to claim their part and get their bond amount
      // The user should recieve all the current funds in the contract
      await expect(lock.connect(user).release(1)).to.be.revertedWith('!expire')

    });
```

## Mitigation steps
In order to ```extendLock``` to work properly, the current implementation  should be changed to:
```diff
function extendLock(
    uint _id,
    address _asset,
    uint _amount,
    uint _period,
    address _sender
) external onlyManager() {
    Bond memory bond = idToBond(_id);
    Bond storage _bond = _idToBond[_id];
    require(bond.owner == _sender, "!owner");
    require(!bond.expired, "Expired");
    require(bond.asset == _asset, "!BondAsset");
    require(bond.pending == 0); //Cannot extend a lock with pending rewards
+   uint currentEpoch = block.timestamp/DAY;
-   require(epoch[bond.asset] == block.timestamp/DAY, "Bad epoch");
    require(epoch[bond.asset] == currentEpoch, "Bad epoch");

+   uint pendingEpochs = bond.expireEpoch - currentEpoch;
+   uint newBondPeriod = pendingEpochs + _period;
+   //In order to respect min bond period when we extend a bon
+   // Next line can be omitted at discretion of the protocol and devs
+   // If it is omitted any created bond would be able to be extended always (except from those with period = 365)
+   require(newBondPeriod >= 7, "MIN PERIOD");

-    require(bond.period+_period <= 365, "MAX PERIOD");
+    require(newBondPeriod <= 365, "MAX PERIOD");
    
    unchecked {
-       uint shares = (bond.amount + _amount) * (bond.period + _period) / 365;
+       uint shares = (bond.amount + _amount) * newBondPeriod / 365;

-       uint expireEpoch = block.timestamp/DAY + bond.period + _period;
+       uint expireEpoch = currentEpoch + newBondPeriod;

        totalShares[bond.asset] += shares-bond.shares;
        _bond.shares = shares;
        _bond.amount += _amount;
        _bond.expireEpoch = expireEpoch;
        _bond.period += _period;
        _bond.mintTime = block.timestamp; 
-       _bond.mintEpoch = epoch[bond.asset];
+       _bond.mintEpoch = currentEpoch;
-       bondPaid[_id][bond.asset] = accRewardsPerShare[bond.asset][epoch[bond.asset]] * _bond.shares / 1e18;
+       bondPaid[_id][bond.asset] = accRewardsPerShare[bond.asset][currentEpoch] * _bond.shares / 1e18;
    }
    emit ExtendLock(_period, _amount, _sender,  _id);
}
```