## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-23

# [Calling `updateNodeRunnerWhitelistStatus` function always reverts](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/378) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L278-L284
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L684-L692
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L426-L492


# Vulnerability details

## Impact
Calling the `updateNodeRunnerWhitelistStatus` function by the DAO supposes to allow the trusted node runners to use and interact with the protocol when `enableWhitelisting` is set to `true`. However, since calling the `updateNodeRunnerWhitelistStatus` function executes `require(isNodeRunnerWhitelisted[_nodeRunner] != isNodeRunnerWhitelisted[_nodeRunner], "Unnecessary update to same status")`, which always reverts, the DAO is unable to whitelist any trusted node runners. Because none of them can be whitelisted, all trusted node runners cannot call functions like `registerBLSPublicKeys` when the whitelisting mode is enabled. As the major functionalities become unavailable, the protocol's usability becomes much limited, and the user experience becomes much degraded.

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L278-L284
```solidity
    function updateNodeRunnerWhitelistStatus(address _nodeRunner, bool isWhitelisted) external onlyDAO {
        require(_nodeRunner != address(0), "Zero address");
        require(isNodeRunnerWhitelisted[_nodeRunner] != isNodeRunnerWhitelisted[_nodeRunner], "Unnecessary update to same status");

        isNodeRunnerWhitelisted[_nodeRunner] = isWhitelisted;
        emit NodeRunnerWhitelistingStatusChanged(_nodeRunner, isWhitelisted);
    }
```

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L684-L692
```solidity
    function _isNodeRunnerValid(address _nodeRunner) internal view returns (bool) {
        require(_nodeRunner != address(0), "Zero address");

        if(enableWhitelisting) {
            require(isNodeRunnerWhitelisted[_nodeRunner] == true, "Invalid node runner");
        }

        return true;
    }
```

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L426-L492
```solidity
    function registerBLSPublicKeys(
        bytes[] calldata _blsPublicKeys,
        bytes[] calldata _blsSignatures,
        address _eoaRepresentative
    ) external payable nonReentrant {
        ...
        require(_isNodeRunnerValid(msg.sender) == true, "Unrecognised node runner");
        ...
    }
```

## Proof of Concept
Please add the following test in `test\foundry\LSDNFactory.t.sol`. This test will pass to demonstrate the described scenario.
```solidity
    function testCallingUpdateNodeRunnerWhitelistStatusFunctionAlwaysReverts() public {
        vm.prank(address(factory));
        manager.updateDAOAddress(admin);

        vm.startPrank(admin);

        vm.expectRevert("Unnecessary update to same status");
        manager.updateNodeRunnerWhitelistStatus(accountOne, true);

        vm.expectRevert("Unnecessary update to same status");
        manager.updateNodeRunnerWhitelistStatus(accountTwo, false);

        vm.stopPrank();
    }
```

## Tools Used
VSCode

## Recommended Mitigation Steps
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L280 can be updated to the following code.

```solidity
        require(isNodeRunnerWhitelisted[_nodeRunner] != isWhitelisted, "Unnecessary update to same status");
```