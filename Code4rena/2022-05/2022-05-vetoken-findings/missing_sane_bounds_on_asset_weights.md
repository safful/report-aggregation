## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Missing sane bounds on asset weights](https://github.com/code-423n4/2022-05-vetoken-findings/issues/192) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeTokenMinter.sol#L41-L46


# Vulnerability details

## Impact
The admin may fat-finger a change, or be malicious, and have the weights be extreme - ranging from zero to `type(uint256).max`, which would cause the booster to pay out unexpected amounts

## Proof of Concept
No bounds checks in the update function:
```solidity
File: contracts/VeTokenMinter.sol   #1

41       function updateveAssetWeight(address veAssetOperator, uint256 newWeight) external onlyOwner {
42           require(operators.contains(veAssetOperator), "not an veAsset operator");
43           totalWeight -= veAssetWeights[veAssetOperator];
44           veAssetWeights[veAssetOperator] = newWeight;
45           totalWeight += newWeight;
46       }
```
https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeTokenMinter.sol#L41-L46

The value is used by the reward contract to determine how much to mint:
```solidity
File: contracts/Booster.sol   #2

598       function rewardClaimed(
599           uint256 _pid,
600           address _address,
601           uint256 _amount
602       ) external returns (bool) {
603           address rewardContract = poolInfo[_pid].veAssetRewards;
604           require(msg.sender == rewardContract || msg.sender == lockRewards, "!auth");
605           ITokenMinter veTokenMinter = ITokenMinter(minter);
606           //calc the amount of veAssetEarned
607           uint256 _veAssetEarned = _amount.mul(veTokenMinter.veAssetWeights(address(this))).div(
608               veTokenMinter.totalWeight()
609           );
610           //mint reward tokens
611           ITokenMinter(minter).mint(_address, _veAssetEarned);
```
https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L598-L611

Wrong values will lead to excessive inflation/deflation

## Tools Used
Code inspection

## Recommended Mitigation Steps
Have sane upper/lower limits on the values


