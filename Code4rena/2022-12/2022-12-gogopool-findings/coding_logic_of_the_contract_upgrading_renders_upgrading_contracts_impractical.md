## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- downgraded by judge
- primary issue
- selected for report
- sponsor confirmed
- fix security (sponsor)
- M-02

# [Coding logic of the contract upgrading renders upgrading contracts impractical](https://github.com/code-423n4/2022-12-gogopool-findings/issues/742) 

# Lines of code

https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ProtocolDAO.sol#L209-L216


# Vulnerability details


## Impact

[Link to original code](https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ProtocolDAO.sol#L209-L216)
```solidity
File: https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ProtocolDAO.sol

205	/// @notice Upgrade a contract by unregistering the existing address, and registring a new address and name
	/// @param newAddr Address of the new contract
	/// @param newName Name of the new contract
	/// @param existingAddr Address of the existing contract to be deleted
209	function upgradeExistingContract(
			address newAddr,
			string memory newName,
			address existingAddr
		) external onlyGuardian {
			registerContract(newAddr, newName);
			unregisterContract(existingAddr);
216	}
```

Function `ProtocolDAO.upgradeExistingContract` handles contract upgrading. However, there are multiple implicaitons of the coding logic in the function, which render the contract upgrading impractical.

**Implication 1**:
The above function `upgradeExistingContract` registers the upgraded contract first, then unregisters the existing contract. This leads to the requirement that the upgraded contract name **must be different from** the existing contract name. Otherwise the updated contract address returned by `Storage.getAddress(keccak256(abi.encodePacked("contract.address", contractName)))` will be `address(0)` (please refer to the below POC Testcase 1). This is because if the upgraded contract uses the original name (i.e. the contract name is not changed), function call  `unregisterContract(existingAddr)` in the `upgradeExistingContract` will override the registered contract address in `Storage` to address(0) due to the use of the same contract name.

Since using the same name after upgrading will run into trouble with current coding logic, a safeguard should be in place to make sure two names are really different. For example, put this statement in the `upgradeExistingContract` function: 
`require(newName != existingName, "Name not changed");`, where `existingName` can be obtained using: 
`string memory existingName = store.getString(keccak256(abi.encodePacked("contract.name", existingAddr)));`.

**Implication 2**:
If we really want a different name for an upgraded contract, we then get into more serious troubles: We have to upgrade other contracts that reference the upgraded contract since contract names are referenced mostly hardcoded (for security considerations). This may lead to a very complicated issue because contracts are cross-referenced.
For example, contract `ClaimNodeOp` references contracts `RewardsPool`, `ProtocolDAO` and `Staking`. At the same time, contract `ClaimNodeOp` is referenced by contracts `RewardsPool` and `Staking`. This means that:
1. If contract `ClaimNodeOp` was upgraded, which means the contract name `ClaimNodeOp` was changed;
2. This requires contracts `RewardsPool` and `Staking` to be upgraded (with new names) in order to correctly reference to newly named `ClaimNodeOp` contract;
3. This further requires those contracts that reference `RewardsPool` or `Staking` to be upgraded in order to correctly reference them;
4. and this further requires those contracts that reference the above upgraded contracts to be upgraded ...
5. This may lead to complicated code management issue and expose new vulnerabilites due to possible incomplete code adaptation.
6. This may render the contracts upgrading impractical.

I rate this issue as high severity due to the fact that:
Contract upgradability is one of the main features of the whole system design (all other contracts are designed upgradable except for `TokenGGP`, `Storage` and `Vault` ). However, the current `upgradeExistingContract` function's coding logic requires the upgraded contract must change its name (refer to the below Testcase 1). This inturn requires to upgrade all relevant cross-referenced contracts (refer to the below Testcase 2). Thus leading to a quite serous code management issue while upgrading contracts, and renders upgrading contracts impractical. 

## Proof of Concept

**Testcase 1**: This testcase demonstrates that current coding logic of upgrading contracts requires: **the upgraded contract must change its name**. Otherwise contract upgrading will run into issue.
Put the below test case in file `ProtocolDAO.t.sol`. The test case demonstrates that `ProtocolDAO.upgradeExistingContract` does not function properly if the upgraded contract does not change the name. That is: the upgraded contract address returned by `Storage.getAddress(keccak256(abi.encodePacked("contract.address", contractName)))` will be `address(0)` if its name unchanged.
```solidity
	function testUpgradeExistingContractWithNameUnchanged() public {
		address addr = randAddress();
		string memory name = "existingName";

		address existingAddr = randAddress();
		string memory existingName = "existingName";

		vm.prank(guardian);
		dao.registerContract(existingAddr, existingName);
		assertEq(store.getBool(keccak256(abi.encodePacked("contract.exists", existingAddr))), true);
		assertEq(store.getAddress(keccak256(abi.encodePacked("contract.address", existingName))), existingAddr);
		assertEq(store.getString(keccak256(abi.encodePacked("contract.name", existingAddr))), existingName);

		vm.prank(guardian);
		//@audit upgrade contract while name unchanged
		dao.upgradeExistingContract(addr, name, existingAddr);
		assertEq(store.getBool(keccak256(abi.encodePacked("contract.exists", addr))), true);
		//@audit the registered address was deleted by function call `PtotocolDAO.unregisterContract(existingAddr)`
		assertEq(store.getAddress(keccak256(abi.encodePacked("contract.address", name))), address(0));
		assertEq(store.getString(keccak256(abi.encodePacked("contract.name", addr))), name);

               //@audit verify that the old contract has been de-registered
		assertEq(store.getBool(keccak256(abi.encodePacked("contract.exists", existingAddr))), false);
		assertEq(store.getAddress(keccak256(abi.encodePacked("contract.address", existingName))), address(0));
		assertEq(store.getString(keccak256(abi.encodePacked("contract.name", existingAddr))), "");
	}
```

**Testcase 2**: This testcase demonstrates that current coding logic of upgrading contracts requires: **in order to upgrade a single contract, all cross-referenced contracts have to be upgraded and change their names**. Otherwise, other contracts will run into issues.
If the upgraded contract does change its name, contract upgrading will succeed. However, other contracts' functions that reference the upgraded contract will fail due to referencing hardcoded contract name. 
The below testcase upgrades contract `ClaimNodeOp` to `ClaimNodeOpV2`. Then, contract `Staking` calls `increaseGGPRewards` which references hardcoded contract name `ClaimNodeOp` in its modifier. The call is failed.

Test steps:
1. Copy contract file `ClaimNodeOp.sol` to `ClaimNodeOpV2.sol`, and rename the contract name from `ClaimNodeOp` to `ClaimNodeOpV2` in file  `ClaimNodeOpV2.sol`;
2. Put the below test file `UpgradeContractIssue.t.sol` under folder `test/unit/`;
3. Run the test.
**Note**: In order to test actual function call after upgrading contract, this testcase upgrades a real contract `ClaimNodeOp`. This is different from the above Testcase 1 which uses a random address to simulate a contract.
```solidity
// File: UpgradeContractIssue.t.sol
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.17;

import "./utils/BaseTest.sol";
import {ClaimNodeOpV2} from "../../contracts/contract/ClaimNodeOpV2.sol";
import {BaseAbstract} from "../../contracts/contract/BaseAbstract.sol";

contract UpgradeContractIssueTest is BaseTest {
	using FixedPointMathLib for uint256;

	address private nodeOp1;

	uint256 internal constant TOTAL_INITIAL_GGP_SUPPLY = 22_500_000 ether;

	function setUp() public override {
		super.setUp();

		nodeOp1 = getActorWithTokens("nodeOp1", MAX_AMT, MAX_AMT);
		vm.prank(nodeOp1);
		ggp.approve(address(staking), MAX_AMT);
		fundGGPRewardsPool();
	}

	function fundGGPRewardsPool() public {
		// guardian is minted 100% of the supply
		vm.startPrank(guardian);
		uint256 rewardsPoolAmt = TOTAL_INITIAL_GGP_SUPPLY.mulWadDown(.20 ether);
		ggp.approve(address(vault), rewardsPoolAmt);
		vault.depositToken("RewardsPool", ggp, rewardsPoolAmt);
		vm.stopPrank();
	}

	function testUpgradeExistingContractWithNameChanged() public {
		
		vm.prank(nodeOp1);
		staking.stakeGGP(10 ether);

                //@audit increase GGPRewards before upgrading contract - succeed
		vm.prank(address(nopClaim));
		staking.increaseGGPRewards(address(nodeOp1), 10 ether);
		assert(staking.getGGPRewards(address(nodeOp1)) == 10 ether);

		//@audit Start to upgrade contract ClaimNodeOp to ClaimNodeOpV2
		
		vm.startPrank(guardian);
		//@audit upgrad contract
		ClaimNodeOpV2 nopClaimV2 = new ClaimNodeOpV2(store, ggp);
		address addr = address(nopClaimV2);
		//@audit contract name must be changed due to the limitation of `upgradeExistingContract` coding logic
		string memory name = "ClaimNodeOpV2";

		//@audit get existing contract ClaimNodeOp info
		address existingAddr = address(nopClaim);
		string memory existingName = "ClaimNodeOp";
		
		//@audit the existing contract should be already registered. Verify its info.
		assertEq(store.getBool(keccak256(abi.encodePacked("contract.exists", existingAddr))), true);
		assertEq(store.getAddress(keccak256(abi.encodePacked("contract.address", existingName))), existingAddr);
		assertEq(store.getString(keccak256(abi.encodePacked("contract.name", existingAddr))), existingName);

                //@audit Upgrade contract
		dao.upgradeExistingContract(addr, name, existingAddr);
		
		//@audit verify that the upgraded contract has correct contract info registered
		assertEq(store.getBool(keccak256(abi.encodePacked("contract.exists", addr))), true);
		assertEq(store.getAddress(keccak256(abi.encodePacked("contract.address", name))), addr);
		assertEq(store.getString(keccak256(abi.encodePacked("contract.name", addr))), name);

		//@audit verify that the old contract has been de-registered
		assertEq(store.getBool(keccak256(abi.encodePacked("contract.exists", existingAddr))), false);
		assertEq(store.getAddress(keccak256(abi.encodePacked("contract.address", existingName))), address(0));
		assertEq(store.getString(keccak256(abi.encodePacked("contract.name", existingAddr))), "");
		vm.stopPrank();

		vm.prank(nodeOp1);
		staking.stakeGGP(10 ether);

                //@audit increase GGPRewards after upgrading contract ClaimNodeOp to ClaimNodeOpV2
		vm.prank(address(nopClaimV2)); //@audit using the upgraded contract
		vm.expectRevert(BaseAbstract.InvalidOrOutdatedContract.selector);
		//@audit revert due to contract Staking using hardcoded contract name "ClaimNodeOp" in the modifier
		staking.increaseGGPRewards(address(nodeOp1), 10 ether);
	}
}

```

## Tools Used
Manual code review

## Recommended Mitigation Steps

1. Upgrading contract does not have to change contranct names especially in such a complicated system wherein contracts are cross-referenced in a hardcoded way. I would suggest not to change contract names when upgrading contracts.
2. In function `upgradeExistingContract` definition, swap fucnction call sequence between `registerContract()` and `unregisterContract()` so that contract names can keep unchanged after upgrading. That is, the modified function would be:
```solidity
File: https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ProtocolDAO.sol

205	/// @notice Upgrade a contract by unregistering the existing address, and registring a new address and name
	/// @param newAddr Address of the new contract
	/// @param newName Name of the new contract
	/// @param existingAddr Address of the existing contract to be deleted
209	function upgradeExistingContract(
			address newAddr,
			string memory newName,  //@audit this `newName` parameter can be removed if upgrading don't change contract name
			address existingAddr
		) external onlyGuardian {
  		unregisterContract(existingAddr);  //@audit unregister the existing contract first
		registerContract(newAddr, newName);  //@audit then register the upgraded contract
216	}
```

**POC of Mitigation**:

After the above recommended mitigation, the below Testcase vefifies that after upgrading contracts, other contract's functions, which reference the hardcoded contract name, can still opetate correctly.

1. Make the above recommended mitigation in function `ProtocolDAO.upgradeExistingContract`;
2. Put the below test file `UpgradeContractImproved.t.sol` under folder `test/unit/`;
3. Run the test.
**Note**: Since we don't change the upgraded contract name, for testing purpose, we just need to create a new contract instance (so that the contract instance address is changed) to simulate the contract upgrading.
```solidity
	// File: UpgradeContractImproved.t.sol
	// SPDX-License-Identifier: GPL-3.0-only
	pragma solidity 0.8.17;

	import "./utils/BaseTest.sol";
	import {ClaimNodeOp} from "../../contracts/contract/ClaimNodeOp.sol";
	import {BaseAbstract} from "../../contracts/contract/BaseAbstract.sol";

	contract UpgradeContractImprovedTest is BaseTest {
		using FixedPointMathLib for uint256;

		address private nodeOp1;

		uint256 internal constant TOTAL_INITIAL_GGP_SUPPLY = 22_500_000 ether;

		function setUp() public override {
			super.setUp();

			nodeOp1 = getActorWithTokens("nodeOp1", MAX_AMT, MAX_AMT);
			vm.prank(nodeOp1);
			ggp.approve(address(staking), MAX_AMT);
			fundGGPRewardsPool();
		}

		function fundGGPRewardsPool() public {
			// guardian is minted 100% of the supply
			vm.startPrank(guardian);
			uint256 rewardsPoolAmt = TOTAL_INITIAL_GGP_SUPPLY.mulWadDown(.20 ether);
			ggp.approve(address(vault), rewardsPoolAmt);
			vault.depositToken("RewardsPool", ggp, rewardsPoolAmt);
			vm.stopPrank();
		}

		function testUpgradeContractCorrectlyWithNameUnChanged() public {
			//@audit increase GGPRewards before upgrading contract - no problem
			vm.prank(nodeOp1);
			staking.stakeGGP(10 ether);

			vm.prank(address(nopClaim));
			staking.increaseGGPRewards(address(nodeOp1), 10 ether);
			assert(staking.getGGPRewards(address(nodeOp1)) == 10 ether);

			//@audit Start to upgrade contract ClaimNodeOp
			vm.startPrank(guardian);
			//@audit upgraded contract by creating a new contract instance
			ClaimNodeOp nopClaimV2 = new ClaimNodeOp(store, ggp);
			address addr = address(nopClaimV2);
			//@audit contract name is not changed!
			string memory name = "ClaimNodeOp";

			//@audit get existing contract info
			address existingAddr = address(nopClaim);
			string memory existingName = "ClaimNodeOp";

			//@audit new contract address is different from the old one
			assertFalse(addr == existingAddr);
			
			//@audit the existing contract should be already registered. Verify its info.
			assertEq(store.getBool(keccak256(abi.encodePacked("contract.exists", existingAddr))), true);
			assertEq(store.getAddress(keccak256(abi.encodePacked("contract.address", existingName))), existingAddr);
			assertEq(store.getString(keccak256(abi.encodePacked("contract.name", existingAddr))), existingName);

                        //@audit Upgrade contract
			dao.upgradeExistingContract(addr, name, existingAddr);
			
			//@audit verify the upgraded contract has correct contract info registered
			assertEq(store.getBool(keccak256(abi.encodePacked("contract.exists", addr))), true);
			assertEq(store.getAddress(keccak256(abi.encodePacked("contract.address", name))), addr);
			assertEq(store.getString(keccak256(abi.encodePacked("contract.name", addr))), name);

			//@audit verify that the old contract has been de-registered
			assertEq(store.getBool(keccak256(abi.encodePacked("contract.exists", existingAddr))), false);
			assertEq(store.getString(keccak256(abi.encodePacked("contract.name", existingAddr))), "");
			//@audit The contract has new address now. Note that: existingName == name
			assertEq(store.getAddress(keccak256(abi.encodePacked("contract.address", existingName))), addr);
			vm.stopPrank();

			//@audit increase GGPRewards after upgrading contract ClaimNodeOp to ClaimNodeOpV2
			vm.prank(nodeOp1);
			staking.stakeGGP(10 ether);

			vm.prank(address(nopClaimV2)); //@audit using the new contract
			//@audit Successfully call the below function with hardcoded contract name "ClaimNodeOp" in the modifier
			staking.increaseGGPRewards(address(nodeOp1), 10 ether);
			//@audit Successfully increased!
			assert(staking.getGGPRewards(address(nodeOp1)) == 20 ether);
		}
	}

```
