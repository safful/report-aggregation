## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [STORAGE COLLISION BETWEEN PROXY AND IMPLEMENTATION (LACK EIP 1967)](https://github.com/code-423n4/2022-03-joyn-findings/issues/108) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreProxy.sol#L9


# Vulnerability details

## Impact
Storage collision because of lack of EIP1967 could cause conflicts and override sensible variables

## Proof of Concept

    contract CoreProxy is Ownable {
           address private immutable _implement;

When you implement proxies, logic and implementation share the same storage layout.    In order to avoid storage conflicts  EIP1967 was proposed.(https://eips.ethereum.org/EIPS/eip-1967)   The idea is to set proxy variables at fixed positions (like  `impl` and `admin` ).  

For example, according to the standard,  the slot for for logic address should be

`0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc` (obtained as `bytes32(uint256(keccak256('eip1967.proxy.implementation')) - 1)`  ).

In this case, for example, as you inherits from `Ownable` the variable _owner is at the first slot and can be overwritten in the implementation.   There is a table at OZ site that explains this scenario more in detail

https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies

section  "Unstructured Storaged Proxies"

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider using EIP1967 



