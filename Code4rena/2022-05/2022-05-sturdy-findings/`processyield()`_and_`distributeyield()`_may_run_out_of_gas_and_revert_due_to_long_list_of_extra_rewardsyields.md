## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`processYield()` and `distributeYield()` may run out of gas and revert due to long list of extra rewards/yields](https://github.com/code-423n4/2022-05-sturdy-findings/issues/70) 

# Lines of code

https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/ConvexCurveLPVault.sol#L105-L110
https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/YieldManager.sol#L129-L136


# Vulnerability details

## Impact
Yields will not be able to be distributed to lenders because attempts to do so will revert

## Proof of Concept
The `processYield()` function loops overall of the extra rewards and transfers them
```solidity
File: smart-contracts/ConvexCurveLPVault.sol   #1

105       uint256 extraRewardsLength = IConvexBaseRewardPool(baseRewardPool).extraRewardsLength();
106       for (uint256 i = 0; i < extraRewardsLength; i++) {
107         address _extraReward = IConvexBaseRewardPool(baseRewardPool).extraRewards(i);
108         address _rewardToken = IRewards(_extraReward).rewardToken();
109         _transferYield(_rewardToken);
110       }
```
https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/ConvexCurveLPVault.sol#L105-L110

There is no guarantee that the tokens involved will be efficient in their use of gas, and there are no upper bounds on the number of extra rewards:

```solidity
    function extraRewardsLength() external view returns (uint256) {
        return extraRewards.length;
    }


    function addExtraReward(address _reward) external returns(bool){
        require(msg.sender == rewardManager, "!authorized");
        require(_reward != address(0),"!reward setting");


        extraRewards.push(_reward);
        return true;
    }
```
https://github.com/convex-eth/platform/blob/main/contracts/contracts/BaseRewardPool.sol#L105-L115

Even if not every extra reward token has a balance, an attacker can sprinkle each one with dust, forcing a transfer by this function

`_getAssetYields()` has a similar issue:
```solidity
File: smart-contracts/YieldManager.sol   #X

129       AssetYield[] memory assetYields = _getAssetYields(exchangedAmount);
130       for (uint256 i = 0; i < assetYields.length; i++) {
131         if (assetYields[i].amount > 0) {
132           uint256 _amount = _convertToStableCoin(assetYields[i].asset, assetYields[i].amount);
133           // 3. deposit Yield to pool for suppliers
134           _depositYield(assetYields[i].asset, _amount);
135         }
136       }
```
https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/YieldManager.sol#L129-L136

## Tools Used
Code inspection

## Recommended Mitigation Steps
Include an offset and length as is done in `YieldManager.distributeYield()`


