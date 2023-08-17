## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- upgraded by judge
- H-03

# [dMute.sol: Attacker can push lock items to victim's array such that redemptions are forever blocked](https://github.com/code-423n4/2023-03-mute-findings/issues/6) 

# Lines of code

https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/dao/dMute.sol#L90-L129
https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/dao/dMute.sol#L135-L139


# Vulnerability details

## Impact
This report deals with how an attacker can abuse the fact that he can lock `MUTE` tokens for any other user and thereby push items to the array of `UserLockInfo` structs of the user.  

There are two functions in the `dMute` contract that iterate over all items in this array ([`RedeemTo`](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/dao/dMute.sol#L90-L129) and [`GetUnderlyingTokens`](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/dao/dMute.sol#L135-L139)).  

Thereby if the attacker pushes sufficient items to the array of a user, he can make the above two functions revert since they require more Gas than the Block Gas Limit.  

According to the `zkSync` documentation the block gas limit is currently 12.5 million ([Link](https://docs.zksync.io/userdocs/tech/#:~:text=With%20the%20current%20block%20gas,can%20process%20over%202000%20TPS.)).  

The attack is of "High" impact for the `RedeemTo` function since this function needs to succeed in order for the user to redeem his `MUTE` tokens.  

The user might have a lot of `MUTE` tokens locked and the attacker can make it such that they can never be redeemed. The attacker cannot gain a profit from this attack, i.e. he cannot steal anything, but due to the possibility of this attack users will not lock their tokens, especially not a lot of them.  

This is all the more severe because the `MuteBond` and `MuteAmplifier` contracts also rely on the locking functionality so those upstream features can also not be used securely.  

In the Mitigation section I will show how the `GetUnderlyingTokens` function can be made to run in $O(1)$ time instead of $O(lock\:array\:length)$.  

The `RedeemTo` function can be made to run in $O(indexes\:array\:length)$ instead of $O(lock\:array\:length)$. The length of the indexes array is determined by the user and simply tells how many locked items to redeem. So there is no possibility of DOS.  

## Proof of Concept
Note: a redemption costs `~7 million Gas` when 1000 items are locked. So when running on the `zkSync` network even 2000 items should be enough. The hardhat tests use a local Ethereum network instead of a fork of `zkSync` so in order to hit `30 million Gas` (which is the Ethereum block gas limit) we need to add more items to the queue.  

You can add the following test to the `dao.ts` test file:  
```javascript
it('Lock DOS', async function () {
    var tx = await muteToken.approve(dMuteToken.address, MaxUint256)


    let lock_time_week = new BigNumber(60 * 60 * 24 * 7);
    let max_lock = lock_time_week.times(52);

    let lock_amount = new BigNumber(1).times(Math.pow(10,2))

    // @audit fill up array
    for(let i=0;i<5000;i++) {
        tx = await dMuteToken.LockTo(lock_amount.toFixed(0), lock_time_week.toFixed(),owner.address)
    }

    await time.increase(60 * 60 * 24 * 7)

    tx = await dMuteToken.Redeem([0])
})
```
It adds `5000` lock items to the array of the `owner` address. When the `owner` then tries to redeem even a single lock the transaction fails due to an out of gas error.  

(Sometimes it reverts with `TransactionExecutionError: Transaction ran out of gas` error sometimes it reverts due to timeout. If you try a few times it should revert with the out of gas error.)  

The amount of `MUTE` tokens that the attacker loses to execute this attack is negligible. As you can see in the test `100 Wei * 5000 = 500,000 Wei` is sufficient (There needs to be some amount of `MUTE` such that the `LockTo` function does not revert). The only real cost comes down to Gas costs which are cheap on `zkSync`.  

## Tools Used
VSCode

## Recommended Mitigation Steps
First for the `GetUnderlyingTokens` function: The contract should keep track of underlying token amounts for each user in a mapping that is updated with every lock / redeem call. The `GetUnderlyingTokens` function then simply needs to return the value from this mapping.  

Secondly, fixing the issue with the `RedeemTo` function is a bit harder. I discussed this with the sponsor and I have been told they don't want this function to require an already sorted `lock_index` array as parameter. So the `lock_index` array can contain indexes in random order.  

This means it must be sorted internally. Depending on the expected length of the `lock_index` array different sorting algorithms may be used. I recommend to use an algorithm like [quick sort](https://gist.github.com/subhodi/b3b86cc13ad2636420963e692a4d896f) to allow for many indexes to be specified at once.  

I will use a placeholder for the sorting algorithm for now so the sponsor may decide which one to use.  

The proposed fixes for both functions are then like this:  
```diff
diff --git a/contracts/dao/dMute.sol b/contracts/dao/dMute.sol
index 59f95b7..11d21fb 100644
--- a/contracts/dao/dMute.sol
+++ b/contracts/dao/dMute.sol
@@ -18,6 +18,7 @@ contract dMute is dSoulBound {
     }
 
     mapping(address => UserLockInfo[]) public _userLocks;
+    mapping(address => uint256) public _amounts;
 
     uint private unlocked = 1;
 
@@ -79,6 +80,7 @@ contract dMute is dSoulBound {
         _mint(to, tokens_to_mint);
 
         _userLocks[to].push(UserLockInfo(_amount, block.timestamp.add(_lock_time), tokens_to_mint));
+        _amounts[to] = _amounts[to] + _amount;
 
         emit LockEvent(to, _amount, tokens_to_mint, _lock_time);
     }
@@ -91,8 +93,14 @@ contract dMute is dSoulBound {
         uint256 total_to_redeem = 0;
         uint256 total_to_burn = 0;
 
-        for(uint256 i; i < lock_index.length; i++){
-          uint256 index = lock_index[i];
+        ///////////////////////////////////////////////
+        //                                           //
+        // sort lock_index array in ascending order //
+        //                                          //
+        //////////////////////////////////////////////
+
+        for(uint256 i = lock_index.length; i > 0; i--){
+          uint256 index = lock_index[i - 1];
           UserLockInfo memory lock_info = _userLocks[msg.sender][index];
 
           require(block.timestamp >= lock_info.time, "dMute::Redeem: INSUFFICIENT_LOCK_TIME");
@@ -102,23 +110,14 @@ contract dMute is dSoulBound {
           total_to_redeem = total_to_redeem.add(lock_info.amount);
           total_to_burn = total_to_burn.add(lock_info.tokens_minted);
 
-          _userLocks[msg.sender][index] = UserLockInfo(0,0,0);
+          _userLocks[msg.sender][index] = _userLocks[msg.sender][_userLocks[msg.sender].length - 1];
+          _userLocks[msg.sender].pop();
         }
 
         require(total_to_redeem > 0, "dMute::Lock: INSUFFICIENT_REDEEM_AMOUNT");
         require(total_to_burn > 0, "dMute::Lock: INSUFFICIENT_BURN_AMOUNT");
 
-
-        for(uint256 i = _userLocks[msg.sender].length; i > 0; i--){
-          UserLockInfo memory lock_info = _userLocks[msg.sender][i - 1];
-
-          // recently redeemed lock, destroy it
-          if(lock_info.time == 0){
-            _userLocks[msg.sender][i - 1] = _userLocks[msg.sender][_userLocks[msg.sender].length - 1];
-            _userLocks[msg.sender].pop();
-          }
-        }
-
+        _amounts[msg.sender] = _amounts[msg.sender] + total_to_redeem;
         //redeem tokens to user
         IERC20(MuteToken).transfer(to, total_to_redeem);
         //burn dMute
@@ -133,8 +132,6 @@ contract dMute is dSoulBound {
     }
 
     function GetUnderlyingTokens(address account) public view returns(uint256 amount) {
-        for(uint256 i; i < _userLocks[account].length; i++){
-          amount = amount.add(_userLocks[account][i].amount);
-        }
+        return _amounts[account];
     }
 }
```



