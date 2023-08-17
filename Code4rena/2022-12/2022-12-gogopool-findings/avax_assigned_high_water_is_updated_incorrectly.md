## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- fix security (sponsor)
- H-01

# [AVAX Assigned High Water is updated incorrectly](https://github.com/code-423n4/2022-12-gogopool-findings/issues/566) 

# Lines of code

https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/MinipoolManager.sol#L374


# Vulnerability details

## Impact

Node operators can manipulate the assigned high water to be higher than the actual.

## Proof of Concept

The protocol rewards node operators according to the `AVAXAssignedHighWater` that is the maximum amount assigned to the specific staker during the reward cycle.
In the function `MinipoolManager.recordStakingStart()`, the `AVAXAssignedHighWater` is updated as below.

```solidity
MinipoolManager.sol
349: 	function recordStakingStart(
350: 		address nodeID,
351: 		bytes32 txID,
352: 		uint256 startTime
353: 	) external {
354: 		int256 minipoolIndex = onlyValidMultisig(nodeID);
355: 		requireValidStateTransition(minipoolIndex, MinipoolStatus.Staking);
356: 		if (startTime > block.timestamp) {
357: 			revert InvalidStartTime();
358: 		}
359:
360: 		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".status")), uint256(MinipoolStatus.Staking));
361: 		setBytes32(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".txID")), txID);
362: 		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".startTime")), startTime);
363:
364: 		// If this is the first of many cycles, set the initialStartTime
365: 		uint256 initialStartTime = getUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".initialStartTime")));
366: 		if (initialStartTime == 0) {
367: 			setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".initialStartTime")), startTime);
368: 		}
369:
370: 		address owner = getAddress(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".owner")));
371: 		uint256 avaxLiquidStakerAmt = getUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".avaxLiquidStakerAmt")));
372: 		Staking staking = Staking(getContractAddress("Staking"));
373: 		if (staking.getAVAXAssignedHighWater(owner) < staking.getAVAXAssigned(owner)) {
374: 			staking.increaseAVAXAssignedHighWater(owner, avaxLiquidStakerAmt);//@audit wrong
375: 		}
376:
377: 		emit MinipoolStatusChanged(nodeID, MinipoolStatus.Staking);
378: 	}
```

In the line #373, if the current assigned AVAX is greater than the owner's `AVAXAssignedHighWater`, it is increased by `avaxLiquidStakerAmt`.
But this is supposed to be updated to `staking.getAVAXAssigned(owner)` rather than being increased by the amount.

Example:
The node operator creates a minipool with 1000AVAX via `createMinipool(nodeID, 2 weeks, delegationFee, 1000*1e18)`.
On creation, the assigned AVAX for the operator will be 1000AVAX.
If the Rialtor calls `recordStakingStart()`, `AVAXAssignedHighWater` will be updated to 1000AVAX.
After the validation finishes, the operator creates another minipool with 1500AVAX this time. Then on `recordStakingStart()`, `AVAXAssignedHighWater` will be updated to 2500AVAX by increasing 1500AVAX because the current assigned AVAX is 1500AVAX which is higher than the current `AVAXAssignedHighWater=1000AVAX`.
This is wrong because the actual highest assigned amount is 1500AVAX.
Note that `AVAXAssignedHighWater` is reset only through the function `calculateAndDistributeRewards` which can be called after `RewardsCycleSeconds=28 days`.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Call `staking.resetAVAXAssignedHighWater(owner)` instead of calling `increaseAVAXAssignedHighWater()`.

```solidity
MinipoolManager.sol
373: 		if (staking.getAVAXAssignedHighWater(owner) < staking.getAVAXAssigned(owner)) {
374: 			staking.resetAVAXAssignedHighWater(owner); //@audit update to the current AVAX assigned
375: 		}
```