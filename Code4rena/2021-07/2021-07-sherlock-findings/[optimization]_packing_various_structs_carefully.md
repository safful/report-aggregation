## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [[Optimization] Packing various structs carefully](https://github.com/code-423n4/2021-07-sherlock-findings/issues/85) 

# Handle

hrkrshnn


# Vulnerability details

## Packing the struct

``` diff
modified   contracts/storage/GovStorage.sol
@@ -14,15 +14,17 @@ library GovStorage {
   struct Base {
     // The address appointed as the govMain entity
     address govMain;
+    // The amount of blocks the cooldown period takes
+    uint40 unstakeCooldown;
+    // The amount of blocks for the window of opportunity of unstaking
+    uint40 unstakeWindow;
+    // Check if the protocol is included in the solution at all
+    uint16 watsonsSherxWeight;
+    // The last block the total amount of rewards were accrued.
     // NOTE: UNUSED
     mapping(bytes32 => address) protocolManagers;
     // Based on the protocol identifier, get the address of the protocol that is able the withdraw balances
     mapping(bytes32 => address) protocolAgents;
-    // The amount of blocks the cooldown period takes
-    uint40 unstakeCooldown;
-    // The amount of blocks for the window of opportunity of unstaking
-    uint40 unstakeWindow;
-    // Check if the protocol is included in the solution at all
     mapping(bytes32 => bool) protocolIsCovered;
     // The array of tokens the accounts are able to stake in
     IERC20[] tokensStaker;
@@ -33,8 +35,6 @@ library GovStorage {
     address watsonsAddress;
     // How much sherX is distributed to this account
     // The max value is uint16(-1), which means 100% of the total SherX minted is allocated to this acocunt
-    uint16 watsonsSherxWeight;
-    // The last block the total amount of rewards were accrued.
     uint40 watsonsSherxLastAccrued;
   }
```

In the current layout, the members `govMain`, `unstakeCooldown`,
`unstakeWindow`, `watsonsSherxWeight` all can be packed to a single slot
or exactly 256 bits. This can save gas if both such elements are read or
written at the same time (please use at least 0.8.2, since it has some
improvements centred around optimizing packed Structs).

In the previous layout:

1.  `govMain` would have a slot of its own.
2.  `unstakeCooldown` and `unstakeWindow` would be packed together in a
    single slot.
3.  `watsonsSherxWeight` and `watsonsSherxLastAccrued` would be packed
    together in a single slot.

Note that gas savings are mainly relevant in the following cases:

1.  Compiler can optimize certain reads and writes to the same slot.
2.  Berlin EIP-2929 based gas accounting, i.e., if the same tx leaves
    one of the slot warm.
3.  Berlin EIP-2930 for access lists. Instead of having to making three
    different slots warm (in the original code), one only has to make
    two slots warm, if necessary.

If none of these applies for your case, this suggestion may be ignored.

## Packing for PoolStorage

``` diff
modified   contracts/storage/PoolStorage.sol
@@ -15,20 +15,35 @@ library PoolStorage {

   struct Base {
     address govPool;
+    // The last block the total amount of rewards were accrued.
+    // Accrueing SherX increases the `unallocatedSherX` variable
+    uint40 sherXLastAccrued;
+    // Protocol debt can only be settled at once for all the protocols at the same time
+    // This variable is the block number the last time all the protocols debt was settled
+    uint40 totalPremiumLastPaid;
+
+    // How much sherX is distributed to stakers of this token
+    // The max value is uint16(-1), which means 100% of the total SherX minted is allocated to this pool
+    uint16 sherXWeight;
     //
     // Staking
     //
     // Indicates if stakers can stake funds in the pool
     bool stakes;
-    // Address of the lockToken. Representing stakes in this pool
-    ILock lockToken;
     // Variable used to calculate the fee when activating the cooldown
     // Max value is uint32(-1) which creates a 100% fee on the withdrawal
     uint32 activateCooldownFee;
+    // Address of the lockToken. Representing stakes in this pool
+    // Indicates if protocol are able to pay premiums with this token
+    // If this value is true, the token is also included as underlying of the SherX
+    bool premiums;
+
+    ILock lockToken;
     // The total amount staked by the stakers in this pool, including value of `firstMoneyOut`
     // if you exclude the `firstMoneyOut` from this value, you get the actual amount of tokens staked
     // This value is also excluding funds deposited in a strategy.
     uint256 stakeBalance;
+
     // All the withdrawals by an account
     // The values of the struct are all deleted if expiry() or unstake() function is called
     mapping(address => UnstakeEntry[]) unstakeEntries;
@@ -39,12 +54,6 @@ library PoolStorage {
     // SherX could be minted before the stakers call the harvest() function
     // Minted SherX that is assigned as reward for the pool will be added to this value
     uint256 unallocatedSherX;
-    // How much sherX is distributed to stakers of this token
-    // The max value is uint16(-1), which means 100% of the total SherX minted is allocated to this pool
-    uint16 sherXWeight;
-    // The last block the total amount of rewards were accrued.
-    // Accrueing SherX increases the `unallocatedSherX` variable
-    uint40 sherXLastAccrued;
     // Non-native variables
     // These variables are used to calculate the right amount of SherX rewards for the token staked
     mapping(address => uint256) sWithdrawn;
@@ -52,9 +61,6 @@ library PoolStorage {
     //
     // Protocol payments
     //
-    // Indicates if protocol are able to pay premiums with this token
-    // If this value is true, the token is also included as underlying of the SherX
-    bool premiums;
     // Storing the protocol token balance based on the protocols bytes32 indentifier
     mapping(bytes32 => uint256) protocolBalance;
     // Storing the protocol premium, the amount of debt the protocol builds up per block.
@@ -62,9 +68,6 @@ library PoolStorage {
     mapping(bytes32 => uint256) protocolPremium;
     // The sum of all the protocol premiums, the total amount of debt that builds up in this token. (per block)
     uint256 totalPremiumPerBlock;
-    // Protocol debt can only be settled at once for all the protocols at the same time
-    // This variable is the block number the last time all the protocols debt was settled
-    uint40 totalPremiumLastPaid;
     // How much token (this) is available for sherX holders
     uint256 sherXUnderlying;
     // Check if the protocol is included in the token pool
```

For the same reasons as before. Taking a quick look at the code, this
change should reduce gas. (Might require 0.8.2, though; there was an
improvement in the optimizer that would apply to packed structs in
storage.)


