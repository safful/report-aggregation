## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Migration fails when all tokens are joined](https://github.com/code-423n4/2022-07-fractional-findings/issues/155) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/f862c14f86adf7de232cd4e9cca6b611e6023b98/src/modules/Migration.sol#L202
https://github.com/code-423n4/2022-07-fractional/blob/f862c14f86adf7de232cd4e9cca6b611e6023b98/src/modules/Migration.sol#L528


# Vulnerability details

## Impact
When `proposal.totalFractions` is equal to the total supply (meaning that all token holders want to participate in a migration), there is a division by zero in `_calculateTotal`.

In contrast to a buyout, where it does not make sense to initiate a buyout if all tokens are held (because there is a dedicated method for that), it does make sense to have a migration that all token holders join. Therefore, this case should be handled.

## Proof Of Concept
```diff
--- a/test/Migration.t.sol
+++ b/test/Migration.t.sol
@@ -238,7 +238,7 @@ contract MigrationTest is TestUtil {
         // Bob joins the proposal
         bob.migrationModule.join{value: 1 ether}(vault, 1, HALF_SUPPLY);
         // Alice joins the proposal
-        alice.migrationModule.join{value: 1 ether}(vault, 1, 1000);
+        alice.migrationModule.join{value: 1 ether}(vault, 1, HALF_SUPPLY);

         vm.warp(proposalPeriod + 1);
         // bob calls commit to kickoff the buyout process
```

## Recommended Mitigation Steps
In such a case, `redeem` can be used instead of starting a buyout.

