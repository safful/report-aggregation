## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [low market bonds/swaps not working after loan is taken from treasury](https://github.com/code-423n4/2022-08-olympus-findings/issues/422) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/TRSRY.sol#L60


# Vulnerability details

## Impact
low market bonds/swaps not working after loan is taken from TRSRY

I am bordering between this being medium and low, but since this is, granted under very unlikely circumstances, is hindering intended transfers to work I am submitting it as medium. That said, I don't think this scenario is very likely since it requires a trusted contract not part of initial release(? no contract in repo used a loan) to take a large loan from TRSRY.

## Proof of Concept
this will cause test to fail on TRANSFER_FAILED due to TRSRY not having the tokens to transfer but `getReserveBalance` says it has, since capacity is determined based on non-existing tokens.

```diff
diff --git a/src/test/policies/Operator.t.sol b/src/test/policies/Operator.t.sol
index e09aec1..5c1e95f 100644
--- a/src/test/policies/Operator.t.sol
+++ b/src/test/policies/Operator.t.sol
@@ -26,6 +26,8 @@ import {OlympusMinter, OHM} from "modules/MINTR.sol";
 import {Operator} from "policies/Operator.sol";
 import {BondCallback} from "policies/BondCallback.sol";
 
+import {ModuleTestFixtureGenerator} from "test/lib/ModuleTestFixtureGenerator.sol";
+
 contract MockOhm is ERC20 {
     constructor(
         string memory _name,
@@ -45,6 +47,7 @@ contract MockOhm is ERC20 {
 // solhint-disable-next-line max-states-count
 contract OperatorTest is Test {
     using FullMath for uint256;
+    using ModuleTestFixtureGenerator for OlympusTreasury;
 
     UserFactory public userCreator;
     address internal alice;
@@ -53,6 +56,9 @@ contract OperatorTest is Test {
     address internal policy;
     address internal heart;
 
+    address public debtor;
+    address public godmode; 
+
     RolesAuthority internal auth;
     BondAggregator internal aggregator;
     BondFixedTermTeller internal teller;
@@ -187,6 +193,18 @@ contract OperatorTest is Test {
 
         reserve.mint(address(treasury), testReserve * 100);
 
+        debtor = treasury.generateFunctionFixture(treasury.getLoan.selector);
+        godmode = treasury.generateGodmodeFixture(type(OlympusTreasury).name);
+        
+        kernel.executeAction(Actions.ActivatePolicy, godmode);
+        kernel.executeAction(Actions.ActivatePolicy, debtor);
+        
+        vm.prank(godmode);
+        treasury.setApprovalFor(debtor, reserve, testReserve * 100);
+
+        vm.prank(debtor);
+        treasury.getLoan(reserve,testReserve*100);
+
         // Approve the operator and bond teller for the tokens to swap
         vm.prank(alice);
         ohm.approve(address(operator), testOhm * 20);
```

same is applicable for low market bonds since they are created based on the same capacity

## Tools Used
vs code + tests

## Recommended Mitigation Steps
determine capacity from actual tokens held by treasury.