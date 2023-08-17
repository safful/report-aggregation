## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- edited-by-warden
- fix security (sponsor)
- M-12

# [Cancellation of minipool may skip MinipoolCancelMoratoriumSeconds checking if it was cancelled before](https://github.com/code-423n4/2022-12-gogopool-findings/issues/519) 

# Lines of code

https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/MinipoolManager.sol#L225-L227
https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/MinipoolManager.sol#L279-L281


# Vulnerability details

## Impact

When canceling a minipool that was canceled before, it may skip MinipoolCancelMoratoriumSeconds checking and allow the user to cancel the minipool immediately.

## Proof of Concept
A user may create a minipool.

```
/// @notice Accept AVAX deposit from node operator to create a Minipool. Node Operator must be staking GGP. Open to public.
	/// @param nodeID 20-byte Avalanche node ID
	/// @param duration Requested validation period in seconds
	/// @param delegationFee Percentage delegation fee in units of ether (2% is 0.2 ether)
	/// @param avaxAssignmentRequest Amount of requested AVAX to be matched for this Minipool
	function createMinipool(
		address nodeID,
		uint256 duration,
		uint256 delegationFee,
		uint256 avaxAssignmentRequest
	) external payable whenNotPaused {
		if (nodeID == address(0)) {
			revert InvalidNodeID();
		}

		ProtocolDAO dao = ProtocolDAO(getContractAddress("ProtocolDAO"));
		if (
			// Current rule is matched funds must be 1:1 nodeOp:LiqStaker
			msg.value != avaxAssignmentRequest ||
			avaxAssignmentRequest > dao.getMinipoolMaxAVAXAssignment() ||
			avaxAssignmentRequest < dao.getMinipoolMinAVAXAssignment()
		) {
			revert InvalidAVAXAssignmentRequest();
		}

		if (msg.value + avaxAssignmentRequest < dao.getMinipoolMinAVAXStakingAmt()) {
			revert InsufficientAVAXForMinipoolCreation();
		}

		Staking staking = Staking(getContractAddress("Staking"));
		staking.increaseMinipoolCount(msg.sender);
		staking.increaseAVAXStake(msg.sender, msg.value);
		staking.increaseAVAXAssigned(msg.sender, avaxAssignmentRequest);

		if (staking.getRewardsStartTime(msg.sender) == 0) {
			staking.setRewardsStartTime(msg.sender, block.timestamp);
		}

		uint256 ratio = staking.getCollateralizationRatio(msg.sender);
		if (ratio < dao.getMinCollateralizationRatio()) {
			revert InsufficientGGPCollateralization();
		}

		// Get a Rialto multisig to assign for this minipool
		MultisigManager multisigManager = MultisigManager(getContractAddress("MultisigManager"));
		address multisig = multisigManager.requireNextActiveMultisig();

		// Create or update a minipool record for nodeID
		// If nodeID exists, only allow overwriting if node is finished or canceled
		// 		(completed its validation period and all rewards paid and processing is complete)
		int256 minipoolIndex = getIndexOf(nodeID);
		if (minipoolIndex != -1) {
			onlyOwner(minipoolIndex);
			requireValidStateTransition(minipoolIndex, MinipoolStatus.Prelaunch);
			resetMinipoolData(minipoolIndex);
			// Also reset initialStartTime as we are starting a whole new validation
			setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".initialStartTime")), 0);

		} else {
			minipoolIndex = int256(getUint(keccak256("minipool.count")));
			// The minipoolIndex is stored 1 greater than actual value. The 1 is subtracted in getIndexOf()
			setUint(keccak256(abi.encodePacked("minipool.index", nodeID)), uint256(minipoolIndex + 1));
			setAddress(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".nodeID")), nodeID);
			addUint(keccak256("minipool.count"), 1);
		}

		// Save the attrs individually in the k/v store
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".status")), uint256(MinipoolStatus.Prelaunch));
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".duration")), duration);
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".delegationFee")), delegationFee);

		setAddress(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".owner")), msg.sender);
		setAddress(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".multisigAddr")), multisig);
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".avaxNodeOpInitialAmt")), msg.value);
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".avaxNodeOpAmt")), msg.value);
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".avaxLiquidStakerAmt")), avaxAssignmentRequest);

		emit MinipoolStatusChanged(nodeID, MinipoolStatus.Prelaunch);

		Vault vault = Vault(getContractAddress("Vault"));
		vault.depositAVAX{value: msg.value}();
	}
```

and after 5 days, the user cancels the minipool

```
	/// @notice Owner of a minipool can cancel the (prelaunch) minipool
	/// @param nodeID 20-byte Avalanche node ID the Owner registered with
	function cancelMinipool(address nodeID) external nonReentrant {
		Staking staking = Staking(getContractAddress("Staking"));
		ProtocolDAO dao = ProtocolDAO(getContractAddress("ProtocolDAO"));
		int256 index = requireValidMinipool(nodeID);
		onlyOwner(index);
		// make sure they meet the wait period requirement
		if (block.timestamp - staking.getRewardsStartTime(msg.sender) < dao.getMinipoolCancelMoratoriumSeconds()) {
			revert CancellationTooEarly();
		}
		_cancelMinipoolAndReturnFunds(nodeID, index);
	}
```

Then, the user recreates the minipool again by calling the same createMinipool function. Then, the user cancels the minipool immediately. The user should not be allowed to cancel the minpool immediately and he should wait for 5 more days.

Added a test unit to MinipoolManager.t.sol

```
	function testMinipoolManager() public {
		address nodeID1 = randAddress();

		vm.startPrank(nodeOp);
		ggp.approve(address(staking), MAX_AMT);
		staking.stakeGGP(100 ether);

		{
			MinipoolManager.Minipool memory mp = createMyMinipool(nodeID1, 1000 ether, 1000 ether, 2 weeks);

			skip(5 days);
			minipoolMgr.cancelMinipool(mp.nodeID); // Must skip 5 days to be executed
		}

		{
			MinipoolManager.Minipool memory mp = createMyMinipool(nodeID1, 1000 ether, 1000 ether, 2 weeks);
			minipoolMgr.cancelMinipool(mp.nodeID); // Do not need 5 days more to be executed which is wrong
		}

		vm.stopPrank();
	}
```

## Tools Used
Manual and added a test unit

## Recommended Mitigation Steps
Change the createMinipool function. Always setRewardsStartTime everytime the minippol is recreated. 

```
/// @notice Accept AVAX deposit from node operator to create a Minipool. Node Operator must be staking GGP. Open to public.
	/// @param nodeID 20-byte Avalanche node ID
	/// @param duration Requested validation period in seconds
	/// @param delegationFee Percentage delegation fee in units of ether (2% is 0.2 ether)
	/// @param avaxAssignmentRequest Amount of requested AVAX to be matched for this Minipool
	function createMinipool(
		address nodeID,
		uint256 duration,
		uint256 delegationFee,
		uint256 avaxAssignmentRequest
	) external payable whenNotPaused {
		if (nodeID == address(0)) {
			revert InvalidNodeID();
		}

		ProtocolDAO dao = ProtocolDAO(getContractAddress("ProtocolDAO"));
		if (
			// Current rule is matched funds must be 1:1 nodeOp:LiqStaker
			msg.value != avaxAssignmentRequest ||
			avaxAssignmentRequest > dao.getMinipoolMaxAVAXAssignment() ||
			avaxAssignmentRequest < dao.getMinipoolMinAVAXAssignment()
		) {
			revert InvalidAVAXAssignmentRequest();
		}

		if (msg.value + avaxAssignmentRequest < dao.getMinipoolMinAVAXStakingAmt()) {
			revert InsufficientAVAXForMinipoolCreation();
		}

		Staking staking = Staking(getContractAddress("Staking"));
		staking.increaseMinipoolCount(msg.sender);
		staking.increaseAVAXStake(msg.sender, msg.value);
		staking.increaseAVAXAssigned(msg.sender, avaxAssignmentRequest);

		--- if (staking.getRewardsStartTime(msg.sender) == 0) {
			staking.setRewardsStartTime(msg.sender, block.timestamp);
		--- }

		uint256 ratio = staking.getCollateralizationRatio(msg.sender);
		if (ratio < dao.getMinCollateralizationRatio()) {
			revert InsufficientGGPCollateralization();
		}

		// Get a Rialto multisig to assign for this minipool
		MultisigManager multisigManager = MultisigManager(getContractAddress("MultisigManager"));
		address multisig = multisigManager.requireNextActiveMultisig();

		// Create or update a minipool record for nodeID
		// If nodeID exists, only allow overwriting if node is finished or canceled
		// 		(completed its validation period and all rewards paid and processing is complete)
		int256 minipoolIndex = getIndexOf(nodeID);
		if (minipoolIndex != -1) {
			requireValidStateTransition(minipoolIndex, MinipoolStatus.Prelaunch);
			resetMinipoolData(minipoolIndex);
			// Also reset initialStartTime as we are starting a whole new validation
			setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".initialStartTime")), 0);
		} else {
			minipoolIndex = int256(getUint(keccak256("minipool.count")));
			// The minipoolIndex is stored 1 greater than actual value. The 1 is subtracted in getIndexOf()
			setUint(keccak256(abi.encodePacked("minipool.index", nodeID)), uint256(minipoolIndex + 1));
			setAddress(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".nodeID")), nodeID);
			addUint(keccak256("minipool.count"), 1);
		}

		// Save the attrs individually in the k/v store
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".status")), uint256(MinipoolStatus.Prelaunch));
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".duration")), duration);
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".delegationFee")), delegationFee);
		setAddress(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".owner")), msg.sender);
		setAddress(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".multisigAddr")), multisig);
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".avaxNodeOpInitialAmt")), msg.value);
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".avaxNodeOpAmt")), msg.value);
		setUint(keccak256(abi.encodePacked("minipool.item", minipoolIndex, ".avaxLiquidStakerAmt")), avaxAssignmentRequest);

		emit MinipoolStatusChanged(nodeID, MinipoolStatus.Prelaunch);

		Vault vault = Vault(getContractAddress("Vault"));
		vault.depositAVAX{value: msg.value}();
	}
```