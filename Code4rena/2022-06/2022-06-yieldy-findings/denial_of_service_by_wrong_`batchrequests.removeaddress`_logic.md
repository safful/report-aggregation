## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Denial of Service by wrong `BatchRequests.removeAddress` logic](https://github.com/code-423n4/2022-06-yieldy-findings/issues/38) 

# Lines of code

https://github.com/code-423n4/2022-06-yieldy/blob/34774d3f5e9275978621fd20af4fe466d195a88b/src/contracts/BatchRequests.sol#L93
https://github.com/code-423n4/2022-06-yieldy/blob/34774d3f5e9275978621fd20af4fe466d195a88b/src/contracts/BatchRequests.sol#L57
https://github.com/code-423n4/2022-06-yieldy/blob/34774d3f5e9275978621fd20af4fe466d195a88b/src/contracts/BatchRequests.sol#L37


# Vulnerability details

## Impact
The `BatchRequests.removeAddress` logic is wrong and it will produce a denial of service.

## Proof of Concept

Removing the element from the array is done using the `delete` statement, but this is not the proper way to remove an entry from an array, it will just set that position to `address(0)`.

Append dummy data:

- `addAddress('0x0000000000000000000000000000000000000001')`
- `addAddress('0x0000000000000000000000000000000000000002')`
- `addAddress('0x0000000000000000000000000000000000000003')`
- `getAddresses()` => `address[]: 0x0000000000000000000000000000000000000001,0x0000000000000000000000000000000000000002,0x0000000000000000000000000000000000000003`

Remove address:
- `removeAddress(0x0000000000000000000000000000000000000002)` (or `0x0000000000000000000000000000000000000003`)
- `getAddresses()` => `address[]: 0x0000000000000000000000000000000000000001,0x0000000000000000000000000000000000000000,0x0000000000000000000000000000000000000003`

Service is denied because it will try to call `canBatchContracts`  to `address(0)`.

## Recommended Mitigation Steps
- To remove an entry in an array you have to use `pop` and move the last element to the removed entry position.

