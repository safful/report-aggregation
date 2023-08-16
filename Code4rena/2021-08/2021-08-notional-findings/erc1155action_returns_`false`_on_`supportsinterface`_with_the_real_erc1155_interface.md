## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [ERC1155Action returns `false` on `supportsInterface` with the real ERC1155 interface](https://github.com/code-423n4/2021-08-notional-findings/issues/61) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
As the return value of `ERC1155.balanceOf` was changed to a signed integer, the `nERC1155Interface` does not implement the `ERC1155` interface and the `supportsInterface` call will return false if people call it with the actual `ERC1155` interface ID.

## Impact
Not all users of the contract might care about the `balance` function and call `supportsInterface` with the original EIP1155 interface.
The contract will still deny the 

## Recommended Mitigation Steps
It is indeed debatable if this contract should be considered implementing ERC1155 and what the correct return value of `supportsInterface(ERC1155.interface)` should be for compatibility.
Users need to be aware that this contract is not standard compliant and the `supportsInterface` call will fail.

