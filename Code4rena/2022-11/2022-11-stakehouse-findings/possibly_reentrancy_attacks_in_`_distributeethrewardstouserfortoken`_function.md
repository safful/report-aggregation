## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-20

# [Possibly reentrancy attacks in `_distributeETHRewardsToUserForToken` function](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/328) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/SyndicateRewardsProcessor.sol#L51-L73
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L146-L167
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L66-L90
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L66-L104
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L110-L143
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L314-L340


# Vulnerability details

### Author: rotcivegaf

### Impact

The root of the problem are in the `_distributeETHRewardsToUserForToken` who makes a call to distribute the ether rewards. With this call the recipient can execute an reentrancy attack calling several times the different function to steal founds or take advantage of other users/protocol

### Proof of Concept

This functions use the `_distributeETHRewardsToUserForToken`:

#### [`beforeTokenTransfer`, **GiantMevAndFeesPool** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L146-L167):

The contract **GiantLP** use the **GiantMevAndFeesPool** contract as [`transferHookProcessor`](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantLP.sol#L14) and when use the functions [`_mint`, `_burn`, `transferFrom` and `transfer` of the ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.7/contracts/token/ERC20/ERC20.sol), the function [`beforeTokenTransfer`](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L146-L167) implemented in the **GiantMevAndFeesPool** bring a possibility to make a reentrancy attack because in the function [`_distributeETHRewardsToUserForToken`](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/SyndicateRewardsProcessor.sol#L51-L73) implemented in the [**GiantMevAndFeesPool** make a `call` to the `_recipient`](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/SyndicateRewardsProcessor.sol#L67-L68)

A contract can call the function `transfer` of **GiantLP** contract several time, transfer an `amount` from and to self, as the update of the [`claimed`](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L203) would not be done until, it is executed the function [`_afterTokenTransfer`](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantLP.sol#L43-L47) of the **GiantLP** contract, the [`due`](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/SyndicateRewardsProcessor.sol#L61) amount calculated in `_distributeETHRewardsToUserForToken` of **SyndicateRewardsProcessor** contract and the `lastInteractedTimestamp` of **GiantLP** contract will be incorrect

### [`withdrawLPTokens`, **GiantPoolBase** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L66-L90):

The possibility of the reentrancy is given when call function [`_onWithdraw`](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L74), this function implemented in [**GiantMevAndFeesPool** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L181-L193) uses `_distributeETHRewardsToUserForToken` and this one call the recipient making the possibility of the reentrancy, breaking the code of [L76-L89](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L76-L89)

### [`batchDepositETHForStaking`, **StakingFundsVault** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L66-L104):

The possibility of the reentrancy is given when call function [`_distributeETHRewardsToUserForToken`](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L88-L93), this function call the recipient making the possibility of the reentrancy, breaking the code of [L76-L89](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L96-L107)

### [`depositETHForStaking`, **StakingFundsVault** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L110-L143):

The possibility of the reentrancy is given when call function [`_distributeETHRewardsToUserForToken`](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L128-L133), this function call the recipient making the possibility of the reentrancy, breaking the code of [L136-L142](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L136-L142)

### [`beforeTokenTransfer`, **StakingFundsVault** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L314-L340):

The possibility of the reentrancy is given when call function `_distributeETHRewardsToUserForToken` in [L333](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L333) and [L337](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L337), this function call the recipient making the possibility of the reentrancy, breaking the code of [L343-L351](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L343-L351)

### Tools Used

Review

### Recommended Mitigation Steps

One possibility its wrap(`deposit`) ether in WETH and transfer as ERC20 token

Another, it's add `nonReentrant` guard to the functions:
- [`beforeTokenTransfer`, **GiantMevAndFeesPool** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L146-L167)
- [`withdrawLPTokens`, **GiantPoolBase** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L66-L90)
- [`batchDepositETHForStaking`, **StakingFundsVault** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L66-L104)
- [`depositETHForStaking`, **StakingFundsVault** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L110-L143)
- [`beforeTokenTransfer`, **StakingFundsVault** contract](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L314-L340)

```diff
File: contracts/liquid-staking/GiantMevAndFeesPool.sol

@@ -143,7 +143,7 @@ contract GiantMevAndFeesPool is ITransferHookProcessor, GiantPoolBase, Syndicate
     }

     /// @notice Allow giant LP token to notify pool about transfers so the claimed amounts can be processed
-    function beforeTokenTransfer(address _from, address _to, uint256) external {
+    function beforeTokenTransfer(address _from, address _to, uint256) external nonReentrant {
         require(msg.sender == address(lpTokenETH), "Caller is not giant LP");
         updateAccumulatedETHPerLP();
```

```diff
File: contracts/liquid-staking/GiantPoolBase.sol

@@ -66,7 +66,7 @@ contract GiantPoolBase is ReentrancyGuard {
     /// @notice Allow a user to chose to withdraw vault LP tokens by burning their giant LP tokens. 1 Giant LP == 1 vault LP
     /// @param _lpTokens List of LP tokens being owned and being withdrawn from the giant pool
     /// @param _amounts List of amounts of giant LP being burnt in exchange for vault LP
-    function withdrawLPTokens(LPToken[] calldata _lpTokens, uint256[] calldata _amounts) external {
+    function withdrawLPTokens(LPToken[] calldata _lpTokens, uint256[] calldata _amounts) external nonReentrant {
         uint256 amountOfTokens = _lpTokens.length;
         require(amountOfTokens > 0, "Empty arrays");
         require(amountOfTokens == _amounts.length, "Inconsistent array lengths");
```

```diff
File: contracts/liquid-staking/StakingFundsVault.sol

@@ -66,7 +66,7 @@ contract StakingFundsVault is
     /// @notice Batch deposit ETH for staking against multiple BLS public keys
     /// @param _blsPublicKeyOfKnots List of BLS public keys being staked
     /// @param _amounts Amounts of ETH being staked for each BLS public key
-    function batchDepositETHForStaking(bytes[] calldata _blsPublicKeyOfKnots, uint256[] calldata _amounts) external payable {
+    function batchDepositETHForStaking(bytes[] calldata _blsPublicKeyOfKnots, uint256[] calldata _amounts) external payable nonReentrant {
         uint256 numOfValidators = _blsPublicKeyOfKnots.length;
         require(numOfValidators > 0, "Empty arrays");
         require(numOfValidators == _amounts.length, "Inconsistent array lengths");

@@ -110,7 +110,7 @@ contract StakingFundsVault is
     /// @notice Deposit ETH against a BLS public key for staking
     /// @param _blsPublicKeyOfKnot BLS public key of validator registered by a node runner
     /// @param _amount Amount of ETH being staked
-    function depositETHForStaking(bytes calldata _blsPublicKeyOfKnot, uint256 _amount) public payable returns (uint256) {
+    function depositETHForStaking(bytes calldata _blsPublicKeyOfKnot, uint256 _amount) public payable nonReentrant returns (uint256) {
         require(liquidStakingNetworkManager.isBLSPublicKeyBanned(_blsPublicKeyOfKnot) == false, "BLS public key is banned or not a part of LSD network");
         require(
             getAccountManager().blsPublicKeyToLifecycleStatus(_blsPublicKeyOfKnot) == IDataStructures.LifecycleStatus.INITIALS_REGISTERED,

@@ -312,7 +312,7 @@ contract StakingFundsVault is
     }

     /// @notice before an LP token is transferred, pay the user any unclaimed ETH rewards
-    function beforeTokenTransfer(address _from, address _to, uint256) external override {
+    function beforeTokenTransfer(address _from, address _to, uint256) external override nonReentrant {
         address syndicate = liquidStakingNetworkManager.syndicate();
         if (syndicate != address(0)) {
             LPToken token = LPToken(msg.sender);
```
