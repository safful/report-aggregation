## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [createProject can be frontrun](https://github.com/code-423n4/2022-03-joyn-findings/issues/26) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/main/core-contracts/contracts/CoreFactory.sol#L70-L77


# Vulnerability details

## Impact

This is dangerous in scam senario because the malicious user can frontrun and become the owner of the collection. As owner, one can withdraw `paymentToken`. (note that _collections.isForSale can be change by frontrunner)

## Proof of Concept

1. Anyone can call `createProject`.

https://github.com/code-423n4/2022-03-joyn/blob/main/core-contracts/contracts/CoreFactory.sol#L70-L77

```solidity
  function createProject(
    string memory _projectId,
    Collection[] memory _collections
  ) external onlyAvailableProject(_projectId) {
    require(
      _collections.length > 0,
      'CoreFactory: should have more at least one collection'
    );
```

## Recommended Mitigation Steps

Two way to mitigate.

1. Consider use white list on project creation.
2. Ask user to sign their address and check the signature against `msg.sender`.  https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol#L102

