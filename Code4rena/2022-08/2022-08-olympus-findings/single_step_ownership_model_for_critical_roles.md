## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed
- edited-by-warden

# [Single step ownership model for critical roles](https://github.com/code-423n4/2022-08-olympus-findings/issues/153) 

# Lines of code

https://github.com/fullyallocated/Default/blob/master/src/Kernel.sol#L192


# Vulnerability details

### Impact

The `executor` and `admin` roles are important administrative roles that can be set to any arbitrary address that the organisation does not control e.g. `address(0).` 

This impact is that the system can no longer be  administered as the `executor` role is the key administrator role for adding, upgrading and removing Kernels, Modules, Policies, Executors and Admins.

Both the `admin` and `executor` roles can be set to an arbitrary address in a single step however it is worse if the `executor` is changed to something like  `address(0)` as no other role can change it back. The `executor` can change the `admin` role but the `admin` cannot change the executor.

Due to the impact I believe this to be of Medium to High severity.

### Proof of Concept

Below is a test demonstrating that the `executor` role can be set to `address(0)` by the current `executor`;

```solidity
function testCorrectness_ChangeExecutorToAddressZero() public {
  // Demonstrate how the executor role can be changed by setting
  // it to the multisig address.
  vm.startPrank(deployer);
  kernel.executeAction(Actions.ChangeExecutor, address(multisig));
  vm.stopPrank();
  assertEq(kernel.executor(), address(multisig));

  // As the current executor set the new executor to be address(0).
  vm.prank(multisig);
  kernel.executeAction(Actions.ChangeExecutor, address(0));
  vm.stopPrank();
  assertEq(kernel.executor(), address(0));
}
```

The `admin` role cannot modify the `executor` so if it is set to a arbitrary address that Olympus does not control it cannot be reset;

```solidity
function testCorrectness_AdminCannotChangeExecutor() public {
  // Demonstrate how the admin role can be changed by setting
  // it to the multisig address.
  vm.startPrank(deployer);
  kernel.executeAction(Actions.ChangeAdmin, address(multisig));
  vm.stopPrank();
  assertEq(kernel.admin(), address(multisig));

  // As the current admin try and change the executor.
  vm.prank(multisig);
  err = abi.encodeWithSignature("Kernel_OnlyExecutor(address)", multisig);
  vm.expectRevert(err);
  kernel.executeAction(Actions.ChangeExecutor, address(0));
  vm.stopPrank();
}
```

### Tools Used

Vim

### Recommended Remediation Steps

The Kernel should implement a two step ownership change for crucial roles such as the `executor` and `admin`. In the first step the ownership change is ‘proposed’ and the address of the new owner (for `executor` or `admin`) is stored in a state variable. As part of the proposal `address(0)` can be checked and a revert take place. In the second step the new owner would then need to ‘accept’ the ownership change by executing a function on the smart contract. 

Furthermore I feel that the `executor` should not not be able to change the `admin` role via `Actions.ChangeAdmin` on [L212](https://github.com/fullyallocated/Default/blob/master/src/Kernel.sol#L212) and the `admin` should be able to set a new `executor`. This would ensure there is proper separation of duties between the `admin` and the `executor` roles.