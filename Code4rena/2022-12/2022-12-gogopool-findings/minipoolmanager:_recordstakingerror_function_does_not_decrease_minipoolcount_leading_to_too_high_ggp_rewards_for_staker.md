## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- fix security (sponsor)
- M-19

# [MinipoolManager: recordStakingError function does not decrease minipoolCount leading to too high GGP rewards for staker](https://github.com/code-423n4/2022-12-gogopool-findings/issues/235) 

# Lines of code

https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/MinipoolManager.sol#L484-L515
https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/ClaimNodeOp.sol#L81-L84
https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/ClaimNodeOp.sol#L51


# Vulnerability details

## Impact
The `MinipoolManager.recordStakingError` function ([https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/MinipoolManager.sol#L484-L515](https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/MinipoolManager.sol#L484-L515)) does not decrease the `minipoolCount` of the staker.  

This means that if a staker has a minipool that encounters an error, his `minipoolCount` can never go to zero again.  

This is bad because the `minipoolCount` is used in `ClaimNodeOp.calculateAndDistributeRewards` to determine if the `rewardsStartTime` of the staker should be reset ([https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/ClaimNodeOp.sol#L81-L84](https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/ClaimNodeOp.sol#L81-L84)).  

Since the `minipoolCount` cannot go to zero, the `rewardsStartTime` will never be reset.  

This means that the staker is immediately eligible for rewards when he creates a minipool again whereas he should have to wait `rewardsEligibilityMinSeconds` before he is eligible (which is 14 days at the moment) ([https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/ClaimNodeOp.sol#L51](https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/ClaimNodeOp.sol#L51)).  

To conclude, failing to decrease the `minipoolCount` allows the staker to earn higher rewards because he is eligible for staking right after he creates a new minipool and does not have to wait again.  

## Proof of Concept
I have created the following test that you can add to the `MinipoolManager.t.sol` file that logs the `minipoolCount` in the `Staking`, `Error` and `Finished` state.  

The `minipoolCount` is always `1` although it should decrease to `0` when `recordStakingError` is called.  

```solidity
function testRecordStakingErrorWrongMinipoolCount() public {
    uint256 duration = 2 weeks;
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

    bytes32 errorCode = "INVALID_NODEID";

    int256 minipoolIndex = minipoolMgr.getIndexOf(mp1.nodeID);

    vm.prank(nodeOp);
    // minipool count when in "Staking" state: 1
    console.log(staking.getMinipoolCount(nodeOp));
    vm.prank(address(rialto));
    minipoolMgr.recordStakingError{value: validationAmt}(mp1.nodeID, errorCode);
    vm.prank(nodeOp);
    // minipool count when in "Error" state: 1
    console.log(staking.getMinipoolCount(nodeOp));

    vm.prank(address(rialto));

    assertEq(vault.balanceOf("MinipoolManager"), depositAmt);

    MinipoolManager.Minipool memory mp1Updated = minipoolMgr.getMinipool(minipoolIndex);

    vm.prank(address(rialto));
    minipoolMgr.finishFailedMinipoolByMultisig(mp1Updated.nodeID);
    MinipoolManager.Minipool memory mp1finished = minipoolMgr.getMinipool(minipoolIndex);
    vm.prank(nodeOp);
    // minipool count when in "Finished" state: 1
    console.log(staking.getMinipoolCount(nodeOp));
}
```

## Tools Used
VSCode

## Recommended Mitigation Steps
You need to simply add the line `staking.decreaseMinipoolCount(owner);` to the `MinipoolManager.recordStakingError` function.  