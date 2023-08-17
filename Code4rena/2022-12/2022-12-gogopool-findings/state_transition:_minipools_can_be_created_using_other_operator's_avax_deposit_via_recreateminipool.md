## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- selected for report
- sponsor confirmed
- fix security (sponsor)
- M-09

# [State Transition: Minipools can be created using other operator's AVAX deposit via recreateMinipool](https://github.com/code-423n4/2022-12-gogopool-findings/issues/569) 

# Lines of code

https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/MinipoolManager.sol#L444


# Vulnerability details

## Impact

Minipools can be created using other operator's AVAX deposit via recreateMinipool

## Proof of Concept

This issue is related to state transition of Minipools.
According to the implementation, the possible states and transitions are as below.

![Imgur](https://imgur.com/ASL235O.jpg)

The Rialto may call `recreateMinipool` when the minipool is in states of `Withdrawable, Finished, Error, Canceled`.
The problem is that these four states are not the same in the sense of holding the node operator's AVAX.
If the state flow has followed `Prelaunch->Launched->Staking->Error`, all the AVAX are still in the vault.
If the state flow has followed `Prelaunch->Launched->Staking->Error->Finished` (last transition by `withdrawMinipoolFunds`), all the AVAX are sent back to the node operator.
So if the Rialto calls `recreateMinipool` for the second case, there are no AVAX deposited from the node operator at that point but there can be AVAX from other mini pools in the state of Prelaunch.
Because there are AVAX in the vault (and these are not managed per staker base), `recreatePool` results in a new mini pool in `Prelaunch` state and it is further possible to go through the normal flow `Prelaunch->Launched->Staking->Withdrawable->Finished`.
And the other minipool that was waiting for launch will not be able to launch because the vault is lack of AVAX.

Below is a test case written to show an example.

```solidity
function testAudit() public {
  uint256 duration = 2 weeks;
  uint256 depositAmt = 1000 ether;
  uint256 avaxAssignmentRequest = 1000 ether;
  uint256 validationAmt = depositAmt + avaxAssignmentRequest;
  uint128 ggpStakeAmt = 200 ether;

  // Node Op 1
  vm.startPrank(nodeOp);
  ggp.approve(address(staking), MAX_AMT);
  staking.stakeGGP(ggpStakeAmt);
  MinipoolManager.Minipool memory mp1 = createMinipool(
    depositAmt,
    avaxAssignmentRequest,
    duration
  );
  vm.stopPrank();

  // Node Op 2
  address nodeOp2 = getActorWithTokens("nodeOp2", MAX_AMT, MAX_AMT);
  vm.startPrank(nodeOp2);
  ggp.approve(address(staking), MAX_AMT);
  staking.stakeGGP(ggpStakeAmt);
  MinipoolManager.Minipool memory mp2 = createMinipool(
    depositAmt,
    avaxAssignmentRequest,
    duration
  );
  vm.stopPrank();

  int256 minipoolIndex = minipoolMgr.getIndexOf(mp1.nodeID);

  address liqStaker1 = getActorWithTokens("liqStaker1", MAX_AMT, MAX_AMT);
  vm.prank(liqStaker1);
  ggAVAX.depositAVAX{ value: MAX_AMT }();

  // Node Op 1: Prelaunch->Launched
  vm.prank(address(rialto));
  minipoolMgr.claimAndInitiateStaking(mp1.nodeID);

  bytes32 txID = keccak256("txid");
  vm.prank(address(rialto));
  // Node Op 1: Launched->Staking
  minipoolMgr.recordStakingStart(mp1.nodeID, txID, block.timestamp);

  vm.prank(address(rialto));
  // Node Op 1: Staking->Error
  bytes32 errorCode = "INVALID_NODEID";
  minipoolMgr.recordStakingError{ value: validationAmt }(mp1.nodeID, errorCode);

  // There are 2*depositAmt AVAX in the pool manager
  assertEq(vault.balanceOf("MinipoolManager"), depositAmt * 2);

  // Node Op 1: Staking->Finished withdrawing funds
  vm.prank(nodeOp);
  minipoolMgr.withdrawMinipoolFunds(mp1.nodeID);
  assertEq(staking.getAVAXStake(nodeOp), 0);
  mp1 = minipoolMgr.getMinipool(minipoolIndex);
  assertEq(mp1.status, uint256(MinipoolStatus.Finished));

  // Rialto recreate a minipool for the mp1
  vm.prank(address(rialto));
  // Node Op 1: Finished -> Prelaunch
  minipoolMgr.recreateMinipool(mp1.nodeID);

  assertEq(vault.balanceOf("MinipoolManager"), depositAmt);

  // Node Op 1: Prelaunch -> Launched
  vm.prank(address(rialto));
  minipoolMgr.claimAndInitiateStaking(mp1.nodeID);

  // Node Op 1: Launched -> Staking
  vm.prank(address(rialto));
  minipoolMgr.recordStakingStart(mp1.nodeID, txID, block.timestamp);

  assertEq(staking.getAVAXStake(nodeOp), 0);
  assertEq(staking.getAVAXAssigned(nodeOp), depositAmt);
  assertEq(staking.getAVAXAssignedHighWater(nodeOp), depositAmt);

  // now try to launch the second operator's pool, it will fail with InsufficientContractBalance
  vm.prank(address(rialto));
  vm.expectRevert(Vault.InsufficientContractBalance.selector);
  minipoolMgr.claimAndInitiateStaking(mp2.nodeID);
}

```

## Tools Used

Manual Review, Foundry

## Recommended Mitigation Steps

Make sure to keep the node operator's deposit status the same for all states that can lead to the same state.
For example, for all states that can transition to Prelaunch, make sure to send the AVAX back to the user and get them back on the call `recreateMiniPool()`.