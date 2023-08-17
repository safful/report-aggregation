## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-08

# [MuteAmplifier.sol: rescueTokens function does not prevent fee tokens from being transferred](https://github.com/code-423n4/2023-03-mute-findings/issues/18) 

# Lines of code

https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L180-L194


# Vulnerability details

## Impact
The [`MuteAmplifier.rescueTokens`](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L180-L194) function allows the `owner` to withdraw tokens that are not meant to be in this contract.  

The contract does protect tokens that ARE meant to be in the contract by not allowing them to be transferred:  

[Link](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L180-L194)  
```solidity
function rescueTokens(address tokenToRescue, address to, uint256 amount) external virtual onlyOwner nonReentrant {
    if (tokenToRescue == lpToken) {
        require(amount <= IERC20(lpToken).balanceOf(address(this)).sub(_totalStakeLpToken),
            "MuteAmplifier::rescueTokens: that Token-Eth belongs to stakers"
        );
    } else if (tokenToRescue == muteToken) {
        if (totalStakers > 0) {
            require(amount <= IERC20(muteToken).balanceOf(address(this)).sub(totalRewards.sub(totalClaimedRewards)),
                "MuteAmplifier::rescueTokens: that muteToken belongs to stakers"
            );
        }
    }


    IERC20(tokenToRescue).transfer(to, amount);
}
```

You can see that `lpToken` and `muteToken` cannot be transferred unless there is an excess amount beyond what is needed by the contract.  

So stakers can be sure that not even the contract `owner` can mess with their stakes.  

The issue is that `lpToken` and `muteToken` are not the only tokens that need to stay in the contract.  

There is also the `fee0` token and the `fee1` token.  

So what can happen is that the `owner` can withdraw `fee0` or `fee1` tokens and users cannot [`payout`](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L275-L311) rewards or [`withdraw`](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L228-L270) their stake because the transfer of `fee0` / `fee1` tokens reverts due to the missing balance.  

Users can of course send `fee0` / `fee1` tokens to the contract to restore the balance. But this is not intended and certainly leaves the user worse off.  

## Proof of Concept
Assume that when an update occurs via the `update` modifier there is an amount of `fee0` tokens claimed:  

[Link](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L88-L121)  
```solidity
    modifier update() {
        if (_mostRecentValueCalcTime == 0) {
            _mostRecentValueCalcTime = firstStakeTime;
        }


        uint256 totalCurrentStake = totalStake();


        if (totalCurrentStake > 0 && _mostRecentValueCalcTime < endTime) {
            uint256 value = 0;
            uint256 sinceLastCalc = block.timestamp.sub(_mostRecentValueCalcTime);
            uint256 perSecondReward = totalRewards.div(endTime.sub(firstStakeTime));


            if (block.timestamp < endTime) {
                value = sinceLastCalc.mul(perSecondReward);
            } else {
                uint256 sinceEndTime = block.timestamp.sub(endTime);
                value = (sinceLastCalc.sub(sinceEndTime)).mul(perSecondReward);
            }


            _totalWeight = _totalWeight.add(value.mul(10**18).div(totalCurrentStake));


            _mostRecentValueCalcTime = block.timestamp;


            (uint fee0, uint fee1) = IMuteSwitchPairDynamic(lpToken).claimFees();


            _totalWeightFee0 = _totalWeightFee0.add(fee0.mul(10**18).div(totalCurrentStake));
            _totalWeightFee1 = _totalWeightFee1.add(fee1.mul(10**18).div(totalCurrentStake));


            totalFees0 = totalFees0.add(fee0);
            totalFees1 = totalFees1.add(fee1);
        }


        _;
    }
```

We can see that `_totalWeightFee0` is updated such that when a user's rewards are calculated the `fee0` tokens will be paid out to the user.  

What happens now is that the `owner` calls `rescueTokens` which transfers the `fee0` tokens out of the contract.  

We can see that when the `fee0` to be paid out to the user is calculated in the `_applyReward` function, the calculation is solely based on `_totalWeightFee0` and does not take into account if the `fee0` tokens still exist in the contract.  

[Link](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L379)  
```solidity
fee0 = lpTokenOut.mul(_totalWeightFee0.sub(_userWeightedFee0[account])).div(10**18);
```

So when the `fee0` tokens are attempted to be transferred to the user that calls `payout` or `withdraw`, the transfer reverts due to insufficient balance.  

## Tools Used
VSCode

## Recommended Mitigation Steps
The `MuteAmplifier.rescueTokens` function should check that only excess `fee0` / `fee1` tokens can be paid out. Such that tokens that will be paid out to stakers need to stay in the contract.  

Fix:  
```diff
diff --git a/contracts/amplifier/MuteAmplifier.sol b/contracts/amplifier/MuteAmplifier.sol
index 9c6fcb5..b154d81 100644
--- a/contracts/amplifier/MuteAmplifier.sol
+++ b/contracts/amplifier/MuteAmplifier.sol
@@ -188,6 +188,18 @@ contract MuteAmplifier is Ownable{
                     "MuteAmplifier::rescueTokens: that muteToken belongs to stakers"
                 );
             }
+        } else if (tokenToRescue == address(IMuteSwitchPairDynamic(lpToken).token0())) {
+            if (totalStakers > 0) {
+                require(amount <= IERC20(IMuteSwitchPairDynamic(lpToken).token0()).balanceOf(address(this)).sub(totalFees0.sub(totalClaimedFees0)),
+                    "MuteAmplifier::rescueTokens: that token belongs to stakers"
+                );
+            }
+        } else if (tokenToRescue == address(IMuteSwitchPairDynamic(lpToken).token1())) {
+            if (totalStakers > 0) {
+                require(amount <= IERC20(IMuteSwitchPairDynamic(lpToken).token1()).balanceOf(address(this)).sub(totalFees1.sub(totalClaimedFees1)),
+                    "MuteAmplifier::rescueTokens: that token belongs to stakers"
+                );
+            }
         }
 
         IERC20(tokenToRescue).transfer(to, amount);
```

The issue discussed in this report also ties in with the fact that the `fee0 <= totalFees0 && fee1 <= totalFees1` check before transferring fee tokens always passes. It does not prevent the scenario that the sponsor wants to prevent which is when there are not enough fee tokens to be transferred the transfer should not block the function.  

So in addition to the above changes I propose to add these changes as well:  

```diff
diff --git a/contracts/amplifier/MuteAmplifier.sol b/contracts/amplifier/MuteAmplifier.sol
index 9c6fcb5..39cd75b 100644
--- a/contracts/amplifier/MuteAmplifier.sol
+++ b/contracts/amplifier/MuteAmplifier.sol
@@ -255,7 +255,7 @@ contract MuteAmplifier is Ownable{
         }
 
         // payout fee0 fee1
-        if ((fee0 > 0 || fee1 > 0) && fee0 <= totalFees0 && fee1 <= totalFees1) {
+        if ((fee0 > 0 || fee1 > 0) && fee0 < IERC20(IMuteSwitchPairDynamic(lpToken).token0()).balanceOf(address(this)) && fee1 < IERC20(IMuteSwitchPairDynamic(lpToken).token1()).balanceOf(address(this))) {
             address(IMuteSwitchPairDynamic(lpToken).token0()).safeTransfer(msg.sender, fee0);
             address(IMuteSwitchPairDynamic(lpToken).token1()).safeTransfer(msg.sender, fee1);
 
@@ -295,7 +295,7 @@ contract MuteAmplifier is Ownable{
         }
 
         // payout fee0 fee1
-        if ((fee0 > 0 || fee1 > 0) && fee0 <= totalFees0 && fee1 <= totalFees1) {
+        if ((fee0 > 0 || fee1 > 0) && fee0 < IERC20(IMuteSwitchPairDynamic(lpToken).token0()).balanceOf(address(this)) && fee1 < IERC20(IMuteSwitchPairDynamic(lpToken).token1()).balanceOf(address(this))) {
             address(IMuteSwitchPairDynamic(lpToken).token0()).safeTransfer(msg.sender, fee0);
             address(IMuteSwitchPairDynamic(lpToken).token1()).safeTransfer(msg.sender, fee1);
 
@@ -331,7 +331,7 @@ contract MuteAmplifier is Ownable{
             }
 
             // payout fee0 fee1
-            if ((fee0 > 0 || fee1 > 0) && fee0 <= totalFees0 && fee1 <= totalFees1) {
+            if ((fee0 > 0 || fee1 > 0) && fee0 < IERC20(IMuteSwitchPairDynamic(lpToken).token0()).balanceOf(address(this)) && fee1 < IERC20(IMuteSwitchPairDynamic(lpToken).token1()).balanceOf(address(this))) {
                 address(IMuteSwitchPairDynamic(lpToken).token0()).safeTransfer(account, fee0);
                 address(IMuteSwitchPairDynamic(lpToken).token1()).safeTransfer(account, fee1);
```