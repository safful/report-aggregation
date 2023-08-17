## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [_baseVestedAmount() and vestedAmount() Return Incorrect Historical Values](https://github.com/code-423n4/2022-09-vtvl-findings/issues/104) 

# Lines of code

https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L183-L187
https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L198


# Vulnerability details

## Description
As the comments in `_baseVestedAmount()` explain, once there is any `_claim.amountWithdrawn`, it will be returned if it is greater than the calculated value `vestAmt`. However, `vestAmt` takes account of time, `_referenceTs`, whereas `_claim.amountWithdrawn` is always the amount withdrawn to date. Therefore, for all historical values below `_claim.amountWithdrawn`, including timestamps before `_claim.startTimestamp` and before `_claim.cliffReleaseTimestamp`, `_claim.amountWithdrawn` will be returned.

## Impact
Given that VTVL is intended to be an accessible platform for use by a wide variety of users, this behaviour does create a security risk. Consider these scenarios:
- A protocol relies on VTVL as an off-the-shelf solution for vesting, but builds other systems (escrow, NFT grants, access, airdrops) that work by checking the value of `vestedAmount()`. Airdrops are especially likely to be interested in historical values. These values would be distorted by how much users have claimed and so would result in an undesirable distribution of resources.
- Even if the above does not occur, consider that VTVL might be passed over as a vesting solution precisely because its historical data is inaccurate.
- A contract could be built that inherits from `VTVLVesting` and attempts to use `_baseVestedAmount()` (which is `internal` and so can be used by inheriting contracts). The inheriting contract might apportion rewards based on historical usage.
- VTVL itself might wish to inherit from `VTVLVesting` in future.

## Proof of Concept
```diff
diff --git a/test/VTVLVesting.ts b/test/VTVLVestingPOC.ts
index bb609fb..073e53f 100644
--- a/test/VTVLVesting.ts
+++ b/test/VTVLVestingPOC.ts
@@ -500,14 +500,37 @@ describe('Revoke Claim', async () => {
   const recipientAddress = await randomAddress();
   const [owner, owner2] = await ethers.getSigners();
 
-  it('allows admin to revoke a valid claim', async () => {
+  it('POC: WITHDRAWN DATA IS UNRELIABLE', async () => {
     const {vestingContract} = await createPrefundedVestingContract({tokenName, tokenSymbol, initialSupplyTokens});
-    await vestingContract.createClaim(recipientAddress, startTimestamp, endTimestamp, cliffReleaseTimestamp, releaseIntervalSecs, linearVestAmount, cliffAmount);
+    const startTimestamp2 = startTimestamp.add(releaseIntervalSecs.mul(100));
+    const endTimestamp2 = endTimestamp.add(releaseIntervalSecs.mul(100));
+    const cliffReleaseTimestamp2 = cliffReleaseTimestamp.add(releaseIntervalSecs.mul(100));
+    await vestingContract.createClaim(owner2.address, startTimestamp2, endTimestamp2, cliffReleaseTimestamp2, releaseIntervalSecs, linearVestAmount, cliffAmount);
+
+    // Fast forward to middle of claim
+    const halfWay = startTimestamp2.toNumber() + (endTimestamp2.toNumber()-startTimestamp2.toNumber())/2;
+    await ethers.provider.send("evm_mine", [halfWay]);
+
+    let vestAmt = await vestingContract.vestedAmount(owner2.address, startTimestamp);
+    console.log("NO WITHDRAWAL, BEFORE VEST START: ",vestAmt.toString());
+    vestAmt = await vestingContract.vestedAmount(owner2.address, startTimestamp2);
+    console.log("NO WITHDRAWAL, AT VEST START: ",vestAmt.toString());
+    vestAmt = await vestingContract.vestedAmount(owner2.address, halfWay);
+    console.log("NO WITHDRAWAL, HALF WAY THROUGH VEST: ",vestAmt.toString());
+    vestAmt = await vestingContract.vestedAmount(owner2.address, endTimestamp2);
+    console.log("NO WITHDRAWAL, AT VEST END: ",vestAmt.toString());
+
+    await (await vestingContract.connect(owner2).withdraw()).wait();
 
-    (await vestingContract.revokeClaim(recipientAddress)).wait();
+    vestAmt = await vestingContract.vestedAmount(owner2.address, startTimestamp);
+    console.log("WITHDRAWAL, BEFORE VEST START: ",vestAmt.toString());
+    vestAmt = await vestingContract.vestedAmount(owner2.address, startTimestamp2);
+    console.log("WITHDRAWAL, AT VEST START: ",vestAmt.toString());
+    vestAmt = await vestingContract.vestedAmount(owner2.address, halfWay);
+    console.log("WITHDRAWAL, HALF WAY THROUGH VEST: ",vestAmt.toString());
+    vestAmt = await vestingContract.vestedAmount(owner2.address, endTimestamp2);
+    console.log("WITHDRAWAL, AT VEST END: ",vestAmt.toString());
 
-    // Make sure it gets reverted
-    expect(await (await vestingContract.getClaim(recipientAddress)).isActive).to.be.equal(false);
   });
 
   it('prohibits a random user from revoking a valid claim', async () => {
```

## Tools Used
Manual Inspection

## Recommended Mitigation Steps
For active claims, there is no reason to consider `_claim.amountWithdrawn`, as it will always have been below or equal to `vestAmt` at any point in time. So only consider `vestAmt` for inactive claims. For them, return the lowest of `vestAmt` and  `_claim.amountWithdrawn`. This will keep the values monotonic with time without distorting the historical values. It will act as though `_claim.amountWithdrawn` was withdrawn and the claim was revoked in the block when `vestAmt` reached `_claim.amountWithdrawn`. That is a distortion, but it is required to provide monotonicity.
