## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Comptroller rewards can be artificially inflated and drained by manipulating [totalStaked - totalFrozen] (or: wrong rewards calculation)](https://github.com/code-423n4/2021-10-union-findings/issues/78) 

# Handle

kenzo


# Vulnerability details

By adding a small of amount of staking to a normal user scenario,
and not approving this small amount as a loan for anybody,
a staker can gain disproportionate amounts of comptroller rewards,
even to the point of draining the contract.
For example:
Stakers A,B,C stake 100, 65, 20, approve it for borrower Z, then staker B stakes an additional 0.07 DAI, and borrower Z borrows 185.
This will result in disproportionate amount of rewards.


As far as I see, this is the main line that causes the inflated amount (*deep breath*):
In calculateRewardsByBlocks, you set:
```
userManagerData.totalStaked = userManagerContract.totalStaked() - userManagerData.totalFrozen;
```
https://github.com/code-423n4/2021-10-union/blob/main/contracts/token/Comptroller.sol#L140
Note that a staker can make this amount very small (depending of course on the current numbers of the protocol).
(A more advanced attacker might diminish the effect of the current numbers of the protocol by initiating fake loans to himself and not paying them.)
This field is then passed to calculateRewards, and passed further to _getInflationIndexNew, and further to _getInflationIndex.
passed to calculateRewards : https://github.com/code-423n4/2021-10-union/blob/main/contracts/token/Comptroller.sol#L167
passed to _getInflationIndexNew : https://github.com/code-423n4/2021-10-union/blob/main/contracts/token/Comptroller.sol#L259
passed to _getInflationIndex : https://github.com/code-423n4/2021-10-union/blob/main/contracts/token/Comptroller.sol#L238
Now we actually use it in the following line (as effectiveAmount):
```
return blockDelta * inflationPerBlock(effectiveAmount).wadDiv(effectiveAmount) + inflationIndex;
```
https://github.com/code-423n4/2021-10-union/blob/main/contracts/token/Comptroller.sol#L315
So 2 things are happening here:
1. mul by ```inflationPerBlock(effectiveAmount)``` - uses the lookup table in Comptroller. This value gets bigger as effectiveAmount gets smaller, and if effectiveAmount is in the area of 10**18, we will get the maximum amount of the lookup.
2. div by ```effectiveAmount``` - as we saw, this can be made small, thereby enlarging the result.
All together, this calculation will be set to ```curInflationIndex``` and then used in the following line:
```
return (curInflationIndex - startInflationIndex).wadMul(effectiveStakeAmount).wadMul(inflationIndex);
```
https://github.com/code-423n4/2021-10-union/blob/main/contracts/token/Comptroller.sol#L263
Note the ```curInflationIndex - startInflationIndex```: per my POC (see below), this can result in a curInflationIndex which is orders of magnitude larger (200x) than startInflationIndex. This creates a huge inflation of rewards.

## Impact
Comptroller rewards can be drained.

## Proof of Concept
See the following script for a POC of reward drainage.
It is based on the scenario in test/integration/testUserManager:
Stakers A,B,C stake 100, 65, 20, and borrower Z borrows 185.
But the difference in my script is that just before borrower Z borrows 185, staker B stakes an additional 0.07 DAI.
(This will be the small amount that is ```totalStaked - totalFrozen```).
Then, we wait 11 blocks to make the loan overdue, call updateOverdueInfo so totalFrozen would be updated, and then staker B calls withdrawRewards.
He ends up with 873 unionTokens out of the 1000 the Comptroller has been seeded with.
And this number can be enlarged by changing the small additional amount that staker B staked.
In this scenario, when calling withdrawRewards, the calculated ```curInflationIndex``` will be 215 WAD, while ```startInflationIndex``` is 1 WAD, and this is the main issue as I understand it. 
File password: "union".
https://pastebin.com/3bJF8mTe


## Tools Used
Manual analysis, hardhat

## Recommended Mitigation Steps
Are you sure that this line should deduct the totalFrozen?
```
userManagerData.totalStaked = userManagerContract.totalStaked() - userManagerData.totalFrozen;
```
https://github.com/code-423n4/2021-10-union/blob/main/contracts/token/Comptroller.sol#L140
Per my tests, if we change it to just 
```
userManagerData.totalStaked = userManagerContract.totalStaked();
```
Then we are getting normal results again and no drainage.
And the var _is_ called just totalStaked...
So maybe this is the change that needs to be made?
But maybe you have a reason to deduct the totalFrozen.
If so, then a mitigation will perhaps be to limit curInflationIndex somehow, maybe by changing the lookup table, or limiting it to a percentage from startInflationIndex ; but even then, there is also the issue of dividing by ```userManagerData.totalStaked``` which can be made quite small as the user has control over that.

