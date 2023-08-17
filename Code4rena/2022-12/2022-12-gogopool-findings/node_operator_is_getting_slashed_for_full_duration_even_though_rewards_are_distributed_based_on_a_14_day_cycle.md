## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- upgraded by judge
- fix security (sponsor)
- H-03

# [node operator is getting slashed for full duration even though rewards are distributed based on a 14 day cycle](https://github.com/code-423n4/2022-12-gogopool-findings/issues/493) 

# Lines of code

https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/MinipoolManager.sol#L673-L675


# Vulnerability details

## Description
A node operator sends in the amount of duration they want to stake for. Behind the scenes Rialto will stake in 14 day cycles and then distribute rewards.

If a node operator doesn't have high enough availability and doesn't get any rewards, the protocol will slash their staked `GGP`. For calculating the expected rewards that are missed however, the full duration is used:

```javascript
File: MinipoolManager.sol

557:	function getExpectedAVAXRewardsAmt(uint256 duration, uint256 avaxAmt) public view returns (uint256) {
558:		ProtocolDAO dao = ProtocolDAO(getContractAddress("ProtocolDAO"));
559:		uint256 rate = dao.getExpectedAVAXRewardsRate();
560:		return (avaxAmt.mulWadDown(rate) * duration) / 365 days; // full duration used when calculating expected reward
561:	}

...

670:	function slash(int256 index) private {

...

673:		uint256 duration = getUint(keccak256(abi.encodePacked("minipool.item", index, ".duration")));
674:		uint256 avaxLiquidStakerAmt = getUint(keccak256(abi.encodePacked("minipool.item", index, ".avaxLiquidStakerAmt")));
675:		uint256 expectedAVAXRewardsAmt = getExpectedAVAXRewardsAmt(duration, avaxLiquidStakerAmt); // full duration
676:		uint256 slashGGPAmt = calculateGGPSlashAmt(expectedAVAXRewardsAmt);
```

This is unfair to the node operator because the expected rewards is from a 14 day cycle.

Also, If they were to be unavailable again, in a later cycle, they would get slashed for the full duration once again.

## Impact
A node operator staking for a long time is getting slashed for an unfairly large amount if they aren't available during a 14 day period.

The protocol also wants node operators to stake in longer periods:
https://multisiglabs.notion.site/Known-Issues-42e2f733daf24893a93ad31100f4cd98
> Team Comment: 

> - This can only be taken advantage of when signing up for 2-4 week validation periods. **Our protocol is incentivizing nodes to sign up for 3-12 month validation periods.** If the team notices this mechanic being abused, Rialto may update its GGP reward calculation to disincentive this behavior.

This slashing amount calculation incentives the node operator to sign up for the shortest period possible and restake themselves to minimize possible losses. 

## Proof of Concept
Test in `MinipoolManager.t.sol`:

```javascript
	function testRecordStakingEndWithSlashHighDuration() public {
		uint256 duration = 365 days;
		uint256 depositAmt = 1000 ether;
		uint256 avaxAssignmentRequest = 1000 ether;
		uint256 validationAmt = depositAmt + avaxAssignmentRequest;
		uint128 ggpStakeAmt = 200 ether;

		vm.startPrank(nodeOp);
		ggp.approve(address(staking), MAX_AMT);
		staking.stakeGGP(ggpStakeAmt);
		MinipoolManager.Minipool memory mp1 = createMinipool(depositAmt, avaxAssignmentRequest, duration);
		vm.stopPrank();

		address liqStaker1 = getActorWithTokens("liqStaker1", MAX_AMT, MAX_AMT);
		vm.prank(liqStaker1);
		ggAVAX.depositAVAX{value: MAX_AMT}();

		vm.prank(address(rialto));
		minipoolMgr.claimAndInitiateStaking(mp1.nodeID);

		bytes32 txID = keccak256("txid");
		vm.prank(address(rialto));
		minipoolMgr.recordStakingStart(mp1.nodeID, txID, block.timestamp);

		skip(2 weeks); // a two week cycle
		
		vm.prank(address(rialto));
		minipoolMgr.recordStakingEnd{value: validationAmt}(mp1.nodeID, block.timestamp, 0 ether);

		assertEq(vault.balanceOf("MinipoolManager"), depositAmt);

		int256 minipoolIndex = minipoolMgr.getIndexOf(mp1.nodeID);
		MinipoolManager.Minipool memory mp1Updated = minipoolMgr.getMinipool(minipoolIndex);
		assertEq(mp1Updated.status, uint256(MinipoolStatus.Withdrawable));
		assertEq(mp1Updated.avaxTotalRewardAmt, 0);
		assertTrue(mp1Updated.endTime != 0);

		assertEq(mp1Updated.avaxNodeOpRewardAmt, 0);
		assertEq(mp1Updated.avaxLiquidStakerRewardAmt, 0);

		assertEq(minipoolMgr.getTotalAVAXLiquidStakerAmt(), 0);

		assertEq(staking.getAVAXAssigned(mp1Updated.owner), 0);
		assertEq(staking.getMinipoolCount(mp1Updated.owner), 0);

		// log slash amount
		console.log("slashedAmount",mp1Updated.ggpSlashAmt);
	}
```

Slashed amount for a `365 days` duration is `100 eth` (10%). However, where they to stake for the minimum time, `14 days` the slashed amount would be only ~`3.8 eth`.

## Tools Used
vs code, forge

## Recommended Mitigation Steps
Either hard code the duration to 14 days for calculating expected rewards or calculate the actual duration using `startTime` and `endTime`. 