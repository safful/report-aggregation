## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- satisfactory
- selected for report
- sponsor confirmed
- M-07

# [MuteAmplifier.sol: multiplier calculation is incorrect which leads to loss of rewards for almost all stakers](https://github.com/code-423n4/2023-03-mute-findings/issues/19) 

# Lines of code

https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L473-L499
https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L366-L388
https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L417-L460


# Vulnerability details

## Impact
This report deals with how the calculation of the `multiplier` in the `MuteAmplifier` contract is not only different from how it is displayed in the [documentation on the website](https://wiki.mute.io/mute/mute-switch/amplifier) but it is also different in a very important way.  

The calculation on the website shows a linear relationship between the `dMUTE / poolSize` ratio and the `APY`. The `dMUTE / poolSize` ratio is also called the `tokenRatio`.  
By "linear" I mean that when a user increases his `tokenRatio` from `0` to `0.1` this has the same effect as when increasing it from `0.9` to `1`.  

The implementation in the `MuteAmplifier.calculateMultiplier` function does not have this linear relationship between `tokenRatio` and `APY`.  
An increase in the `tokenRatio` from `0` to `0.1` is worth much less than an increase from `0.9` to `1`.  

As we will see this means that all stakers that do not have a `tokenRatio` of exactly equal `0` or exactly equal `1` lose out on rewards that they should receive according to the documentation.  

I estimate this to be of "High" severity because the issue affects nearly all stakers and results in a partial loss of rewards.  

## Proof of Concept
Let's first look at the `multiplier` calculation from the [documentation](https://wiki.mute.io/mute/mute-switch/amplifier):  

![multiplier](https://user-images.githubusercontent.com/118979828/229354171-7308e84d-435a-4e41-989d-5c37b5ad0faa.png)

![multiplier_example](https://user-images.githubusercontent.com/118979828/229354217-2277ae01-8088-479a-8691-6bf4ad62387a.png)

The example calculation shows that the amount that is added to $APY_{base}$ (5%) is scaled linearly by the $\dfrac{user_{dmute}}{pool_{rewards}}$ ratio which I called `tokenRatio` above.  

This means that when a user increases his `tokenRatio` from say `0` to `0.1` he gets the same additional reward as when he increases the `tokenRatio` from say `0.9` to `1`.  

Let's now look at how the reward and thereby the `multiplier` is calculated in the code.  

The first step is to calculate the `multiplier` which happens in the `MuteAmplifier.calculateMultiplier` function:  

[Link](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L473-L499)  
```solidity
function calculateMultiplier(address account, bool enforce) public view returns (uint256) {
    require(account != address(0), "MuteAmplifier::calculateMultiplier: missing account");


    uint256 accountDTokenValue;


    // zkSync block.number = L1 batch number. This at times is the same for a few minutes. To avoid breaking the call to the dMute contract
    // we take the previous block into account
    uint256 staked_block =  _userStakedBlock[account] == block.number ? _userStakedBlock[account] - 1 : _userStakedBlock[account];


    if(staked_block != 0 && enforce)
        accountDTokenValue = IDMute(dToken).getPriorVotes(account, staked_block);
    else
        accountDTokenValue = IDMute(dToken).getPriorVotes(account, block.number - 1);


    if(accountDTokenValue == 0){
        return _stakeDivisor;
    }


    uint256 stakeDifference = _stakeDivisor.sub(10 ** 18);


    // ratio of dMute holdings to pool
    uint256 tokenRatio = accountDTokenValue.mul(10**18).div(totalRewards);


    stakeDifference = stakeDifference.mul(clamp_value(tokenRatio, 10**18)).div(10**18);


    return _stakeDivisor.sub(stakeDifference);
}
```

The `multiplier` that is returned is then used to calculate the `reward`:  

[Link](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L371)  
```solidity
reward = lpTokenOut.mul(_totalWeight.sub(_userWeighted[account])).div(calculateMultiplier(account, true));
```

Let's write the formula in a more readable form:  

$\dfrac{lpTokenOut * weightDifference}{stakeDivisor - tokenRatio * (stakeDivisor - 1)}$

$stakeDivisor$ can be any number $>=1$ and has the purpose of determining the percentage of rewards a user with $tokenRatio=0$ gets.  

For the sake of this argument we can assume that all numbers except $tokenRatio$ are constant.  

Let's just say $stakeDivisor=2$ which means that a user with $tokenRatio=0§ would receive $\dfrac{1}{2}=50\%$ of the maximum rewards.  

Further let's say that $lpTokenOut * weightDifference = 1$, so 100% of the possible rewards would be $1$.  

We can then write the formula like this:  

$\dfrac{1}{2 - tokenRatio}$

So let's compare the calculation from the documentation with the calculation from the code by looking at a plot:  

![functions](https://user-images.githubusercontent.com/118979828/229356435-49bcba3c-968e-4613-aded-89c2d0145905.png)

![plot](https://user-images.githubusercontent.com/118979828/229355365-9236bf2c-6bd5-4732-97b9-2d008043b7cc.png)  

**x-axis: tokenRatio**  
**y-axis: percentage of maximum rewards**  


We can see that the green curve is non-linear and below the blue curve.  

So the rewards as calculated in the code are too low.  

## Tools Used
VSCode


## Recommended Mitigation Steps
I recommend to change the reward calculation to this:  

$(lpTokenOut * weightDifference) * (percentage_{min} + clamp(\dfrac{user_{dmute}}{pool_{rewards}},1) * (1 - percentage_{min}))$

Instead of setting the `stakeDivisor` upon initialization, the `percentageMin` should be set which can be in the interval `[0,1e18]`.  

Fix:  
```diff
diff --git a/contracts/amplifier/MuteAmplifier.sol b/contracts/amplifier/MuteAmplifier.sol
index 9c6fcb5..1c86f5c 100644
--- a/contracts/amplifier/MuteAmplifier.sol
+++ b/contracts/amplifier/MuteAmplifier.sol
@@ -48,7 +48,7 @@ contract MuteAmplifier is Ownable{
 
     uint256 private _mostRecentValueCalcTime; // latest update modifier timestamp
 
-    uint256 public _stakeDivisor; // divisor set in place for modification of reward boost
+    uint256 public _percentageMin; // minimum percentage set in place for modification of reward boost
 
     uint256 public management_fee; // lp withdrawal fee
     address public treasury; // address that receives the lp withdrawal fee
@@ -131,8 +131,8 @@ contract MuteAmplifier is Ownable{
      *  @param _mgmt_fee uint
      *  @param _treasury address
      */
-    constructor (address _lpToken, address _muteToken, address _dToken, uint256 divisor, uint256 _mgmt_fee, address _treasury) {
-        require(divisor >= 10 ** 18, "MuteAmplifier: invalid _stakeDivisor");
+    constructor (address _lpToken, address _muteToken, address _dToken, uint256 percentageMin, uint256 _mgmt_fee, address _treasury) {
+        require(_percentageMin <= 10 ** 18, "MuteAmplifier: invalid _percentageMin");
         require(_lpToken != address(0), "MuteAmplifier: invalid lpToken");
         require(_muteToken != address(0), "MuteAmplifier: invalid muteToken");
         require(_dToken != address(0), "MuteAmplifier: invalid dToken");
@@ -142,7 +142,7 @@ contract MuteAmplifier is Ownable{
         lpToken = _lpToken;
         muteToken = _muteToken;
         dToken = _dToken;
-        _stakeDivisor = divisor;
+        _percentageMin = percentageMin;
         management_fee = _mgmt_fee; //bps 10k
         treasury = _treasury;
 
@@ -368,7 +368,7 @@ contract MuteAmplifier is Ownable{
         require(lpTokenOut > 0, "MuteAmplifier::_applyReward: no coins staked");
 
         // current rewards based on multiplier
-        reward = lpTokenOut.mul(_totalWeight.sub(_userWeighted[account])).div(calculateMultiplier(account, true));
+        reward = lpTokenOut.mul(_totalWeight.sub(_userWeighted[account])).div(10 ** 18).mul(calculateMultiplier(account, true)).div(10 ** 18);
         // max possible rewards
         remainder = lpTokenOut.mul(_totalWeight.sub(_userWeighted[account])).div(10**18);
         // calculate left over rewards
@@ -442,7 +442,7 @@ contract MuteAmplifier is Ownable{
             uint256 _totalWeightFee1Local = _totalWeightFee1.add(fee1.mul(10**18).div(totalCurrentStake));
 
             // current rewards based on multiplier
-            info.currentReward = totalUserStake(user).mul(totWeightLocal.sub(_userWeighted[user])).div(info.multiplier_last);
+            info.currentReward = totalUserStake(user).mul(totWeightLocal.sub(_userWeighted[user])).div(10 ** 18).mul(info.multiplier_last).div(10 ** 18);
             // add back any accumulated rewards
             info.currentReward = info.currentReward.add(_userAccumulated[user]);
 
@@ -452,7 +452,7 @@ contract MuteAmplifier is Ownable{
 
         } else {
           // current rewards based on multiplier
-          info.currentReward = totalUserStake(user).mul(_totalWeight.sub(_userWeighted[user])).div(info.multiplier_last);
+          info.currentReward = totalUserStake(user).mul(_totalWeight.sub(_userWeighted[user])).div(10 ** 18).mul(info.multiplier_last).div(10 ** 18);
           // add back any accumulated rewards
           info.currentReward = info.currentReward.add(_userAccumulated[user]);
         }
@@ -485,17 +485,17 @@ contract MuteAmplifier is Ownable{
           accountDTokenValue = IDMute(dToken).getPriorVotes(account, block.number - 1);
 
         if(accountDTokenValue == 0){
-          return _stakeDivisor;
+          return _percentageMin;
         }
 
-        uint256 stakeDifference = _stakeDivisor.sub(10 ** 18);
+        uint256 percentageDifference = (uint256(10 ** 18)).sub(_percentageMin);
 
         // ratio of dMute holdings to pool
         uint256 tokenRatio = accountDTokenValue.mul(10**18).div(totalRewards);
 
-        stakeDifference = stakeDifference.mul(clamp_value(tokenRatio, 10**18)).div(10**18);
+        uint256 additionalPercentage = percentageDifference.mul(clamp_value(tokenRatio, 10**18)).div(10**18);
 
-        return _stakeDivisor.sub(stakeDifference);
+        return _percentageMin.add(additionalPercentage);
     }
```
