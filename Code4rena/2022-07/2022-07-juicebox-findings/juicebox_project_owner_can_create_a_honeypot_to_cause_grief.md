## Tags

- bug
- documentation
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- valid

# [Juicebox project owner can create a honeypot to cause grief](https://github.com/code-423n4/2022-07-juicebox-findings/issues/170) 

# Lines of code

https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBController.sol#L760
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSplitsStore.sol#L147


# Vulnerability details

## Impact
In a Juicebox project the project owner (or anyone that they approve) can set splits. These splits are details of the token distributions to other addresses in response to contributions to the project.

At the moment the `SPLITS_TOTAL_PERCENT = 1_000_000_000`. This means that the project owner could theoretically add 1 billion different splits, each with a percent value of 1. Of course, this would require too much gas, but the idea stands. A project owner could honeypot users by creating a project with the `MAX_RESERVED_RATE` reserved rate, and setting a large percentage split for the `msg.sender` who calls `distributeReservedTokensOf` in `JBController.sol`. The project owner could then fund the project with a series of large payments to ensure that the reserved amount was sufficiently large to entice a user to call `distributeReservedTokensOf` in the belief that they will be obtaining a large percentage of the reserve.

However, when a user calls this method they will hit the block gas limit and will have spent a large amount of ETH on gas, without receiving any of their expected split.

I consider this to be of high severity since user assets (in the form of gas) can be permanently lost without any loss to the project owner/griefer.

## Proof of Concept
The key behaviour we need to prove is that it's possible to set more splits before hitting the block gas limit than it is to distribute reward tokens over the same number of splits. If this is true, the project owner will be able to set a number of splits that will always make the `distributeReservedTokensOf` hit the block gas limit, and hence grief the caller.

This can be demonstrated by modifying the existing test cases. From some basic testing I have found that calling `distributeReservedTokensOf` hits the block gas limit when there are at least 389 splits, but for the same split count the project owner can successfully call `set` without hitting the block gas limit.

```
diff --git a/test/jb_controller/distribute_reserved_token_of.test.js b/test/jb_controller/distribute_reserved
_token_of.test.js
index 2f964d8..6cfd645 100644
--- a/test/jb_controller/distribute_reserved_token_of.test.js
+++ b/test/jb_controller/distribute_reserved_token_of.test.js
@@ -119,10 +119,15 @@ describe('JBController::distributeReservedTokensOf(...)', function () {
     const { addrs, projectOwner, jbController, mockJbTokenStore, mockSplitsStore, timestamp } =
       await setup();
     const caller = addrs[0];
-    const splitsBeneficiariesAddresses = [addrs[1], addrs[2]].map((signer) => signer.address);
+    let addressList = [addrs[1], addrs[2]];
+    for (let i = 1; i < 389; i++) {
+      addressList.push(addrs[1]);
+    }
+
+    const splitsBeneficiariesAddresses = addressList.map((signer) => signer.address);

     const splits = makeSplits({
-      count: 2,
+      count: 389,
       beneficiary: splitsBeneficiariesAddresses,
       preferClaimed: true,
     });
diff --git a/test/jb_splits_store/set.test.js b/test/jb_splits_store/set.test.js
index 3dd0331..5992957 100644
--- a/test/jb_splits_store/set.test.js
+++ b/test/jb_splits_store/set.test.js
@@ -54,7 +54,7 @@ describe('JBSplitsStore::set(...)', function () {
     };
   }

-  function makeSplits(beneficiaryAddress, count = 4) {
+  function makeSplits(beneficiaryAddress, count = 389) {
     let splits = [];
     for (let i = 0; i < count; i++) {
       splits.push({
```

## Tools Used
VSCode & Hardhat

## Recommended Mitigation Steps
For `JBSplit` objects there should be a minimum percentage for each split when calling `set`. Furthermore, it would probably be wise to prevent duplicate beneficiaries, but I have omitted that in the below recommendation for clarity. Below is a suggested diff. I've arbitrarily set a minimum percentage of 10,000 but given the PoC the min percentage should be conservatively set to ensure no more than 389 splits can be created (I would probably suggest a cap of max 100 splits per group).

```
diff --git a/contracts/JBSplitsStore.sol b/contracts/JBSplitsStore.sol
index d61cca2..429d78a 100644
--- a/contracts/JBSplitsStore.sol
+++ b/contracts/JBSplitsStore.sol
@@ -227,8 +227,8 @@ contract JBSplitsStore is IJBSplitsStore, JBOperatable {
     uint256 _percentTotal = 0;

     for (uint256 _i = 0; _i < _splits.length; _i++) {
-      // The percent should be greater than 0.
-      if (_splits[_i].percent == 0) revert INVALID_SPLIT_PERCENT();
+      // The percent should be greater than or equal to 10000.
+      if (_splits[_i].percent < JBConstants.MIN_SPLIT_PERCENT) revert INVALID_SPLIT_PERCENT();

       // ProjectId should be within a uint56
       if (_splits[_i].projectId > type(uint56).max) revert INVALID_PROJECT_ID();
diff --git a/contracts/libraries/JBConstants.sol b/contracts/libraries/JBConstants.sol
index 9a418f2..afb5f23 100644
--- a/contracts/libraries/JBConstants.sol
+++ b/contracts/libraries/JBConstants.sol
@@ -10,6 +10,7 @@ library JBConstants {
   uint256 public constant MAX_REDEMPTION_RATE = 10000;
   uint256 public constant MAX_DISCOUNT_RATE = 1000000000;
   uint256 public constant SPLITS_TOTAL_PERCENT = 1000000000;
+  uint256 public constant MIN_SPLIT_PERCENT = 10000;
   uint256 public constant MAX_FEE = 1000000000;
   uint256 public constant MAX_FEE_DISCOUNT = 1000000000;
 }
```

An alternative to setting a minimum percentage would be to have a check on the length of the splits array and capping that at a sensible value. In this instance a project owner could still set low percentages per split, however I don't personally see the value in being able to set a value of 1 (to receive 1 billionth of the reserve).



