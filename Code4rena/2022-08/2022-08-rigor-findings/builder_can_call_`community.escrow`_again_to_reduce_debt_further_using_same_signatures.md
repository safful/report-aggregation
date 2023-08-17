## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- valid

# [Builder can call `Community.escrow` again to reduce debt further using same signatures](https://github.com/code-423n4/2022-08-rigor-findings/issues/161) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Community.sol#L509


# Vulnerability details

## Impact

Since there is no nonce in the data decoded at the beginning of function `escrow`, a builder can call the function multiple times reducing their debt as much as they wish.

## Proof of Concept

- A builder has a debt of $50,000
- A lender, a builder, and an escrow agent all ~~enter a bar~~ sign a message that will reduce the debt of the builder by $5,000, upon receipt of physical cash.
- Function `escrow` is called and debt is reduced to $45,000.
- The builder, using the same `_data` and `_signature` then calls `escrow` a further 9 times reducing their debt to zero.

## Recommended Mitigation Steps

1. Similar to function `publishProject`, add a new field into the [ProjectDetails](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/interfaces/ICommunity.sol#L19-L32) struct called `escrowNonce`.

2. Modify function `escrow` to check this nonce and update it after the debt has been reduced.

See the diff below for full changes.

```diff
diff --git a/contracts/Community.sol b/contracts/Community.sol
index 1585670..b834d0e 100644
--- a/contracts/Community.sol
+++ b/contracts/Community.sol
@@ -15,7 +15,7 @@ import {SignatureDecoder} from "./libraries/SignatureDecoder.sol";

 /**
  * @title Community Contract for HomeFi v2.5.0
-
+
  * @notice Module for coordinating lending groups on HomeFi protocol
  */
 contract Community is
@@ -520,10 +520,11 @@ contract Community is
             address _agent,
             address _project,
             uint256 _repayAmount,
+            uint256 _escrowNonce,
             bytes memory _details
         ) = abi.decode(
                 _data,
-                (uint256, address, address, address, address, uint256, bytes)
+                (uint256, address, address, address, address, uint256, uint256, bytes)
             );

         // Compute hash from bytes
@@ -540,6 +541,12 @@ contract Community is
             _lender == _communities[_communityID].owner,
             "Community::!Owner"
         );
+        ProjectDetails storage _communityProject =
+          _communities[_communityID].projectDetails[_project];
+        require(
+            _escrowNonce == _communityProject.escrowNonce,
+            "Community::invalid escrowNonce"
+        );

         // check signatures
         checkSignatureValidity(_lender, _hash, _signature, 0); // must be lender
@@ -548,6 +555,7 @@ contract Community is

         // Internal call to reduce debt
         _reduceDebt(_communityID, _project, _repayAmount, _details);
+        _communityProject.escrowNonce = _communityProject.escrowNonce + 1;
         emit DebtReducedByEscrow(_agent);
     }

diff --git a/contracts/interfaces/ICommunity.sol b/contracts/interfaces/ICommunity.sol
index c45bbf0..652f51c 100644
--- a/contracts/interfaces/ICommunity.sol
+++ b/contracts/interfaces/ICommunity.sol
@@ -29,6 +29,7 @@ interface ICommunity {
         uint256 lentAmount; // current principal lent to project (needs to be repaid by project's builder)
         uint256 interest; // total accrued interest on `lentAmount`
         uint256 lastTimestamp; // timestamp when last lending / repayment was made
+        uint256 escrowNonce; // signing nonce to use when reducing debt by escrow
     }
```