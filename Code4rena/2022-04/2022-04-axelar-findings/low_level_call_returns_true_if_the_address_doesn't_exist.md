## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Low level call returns true if the address doesn't exist](https://github.com/code-423n4/2022-04-axelar-findings/issues/11) 

# Lines of code

https://github.com/code-423n4/2022-04-axelar/blob/dee2f2d352e8f20f20027977d511b19bfcca23a3/src/AxelarGateway.sol#L545-L548
https://github.com/code-423n4/2022-04-axelar/blob/dee2f2d352e8f20f20027977d511b19bfcca23a3/src/AxelarGatewayProxy.sol#L16-L24


# Vulnerability details

## Impact
As written in the [solidity documentation](https://docs.soliditylang.org/en/develop/control-structures.html#error-handling-assert-require-revert-and-exceptions), the low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed. 

## Proof of Concept
The low-level functions `call` and `delegatecall` are used in some places in the code and it can be problematic. For example, in the `_callERC20Token` of the `AxelarGateway` contract there is a low level call in order to call the ERC20 functions, but if the given `tokenAddress` doesn't exist `success` will be equal to true and the function will return true and the code execution will be continued like the call was successful. 
```sol
function _callERC20Token(address tokenAddress, bytes memory callData) internal returns (bool) {
    (bool success, bytes memory returnData) = tokenAddress.call(callData);
    return success && (returnData.length == uint256(0) || abi.decode(returnData, (bool)));
}
```
Another place that this can happen is in `AxelarGatewayProxy`'s constructor
```sol
constructor(address gatewayImplementation, bytes memory params) {
    _setAddress(KEY_IMPLEMENTATION, gatewayImplementation);

   (bool success, ) = gatewayImplementation.delegatecall(
       abi.encodeWithSelector(IAxelarGateway.setup.selector, params)
   );

    if (!success) revert SetupFailed();
}
```
If the `gatewayImplementation` address doesn't exist, the delegate call will return true and the function won't revert.

## Tools Used
Remix, VS Code

## Recommended Mitigation Steps
Check before any low-level call that the address actually exists, for example before the low level call in the callERC20 function you can check that the address is a contract by checking its code size.

