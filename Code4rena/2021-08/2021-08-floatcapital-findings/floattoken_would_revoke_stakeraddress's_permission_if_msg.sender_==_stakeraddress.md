## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [FloatToken would revoke stakerAddress's permission if msg.sender == stakerAddress](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/36) 

# Handle

jonah1005


# Vulnerability details

## Impact
FloatToken would revoke staker's permission if msg.sender == stakerAddress.
In `initializeFloatToken` the contract would first grant roles to `stakerAddress` and than revoke`msg.sender`'s permissions. The contract would be left with no privileged address if stakerAddress == msg.sender.


## Proof of Concept
https://github.com/code-423n4/2021-08-floatcapital/blob/main/contracts/contracts/FloatToken.sol#L21-L35

## Tools Used
None
## Recommended Mitigation Steps

```solidity
  function initializeFloatToken(
    string calldata name,
    string calldata symbol,
    address stakerAddress
  ) external initializer {
    initialize(name, symbol);

    renounceRole(DEFAULT_ADMIN_ROLE, msg.sender);
    renounceRole(MINTER_ROLE, msg.sender);
    renounceRole(PAUSER_ROLE, msg.sender);
  
    _setupRole(DEFAULT_ADMIN_ROLE, stakerAddress);
    _setupRole(MINTER_ROLE, stakerAddress);
    _setupRole(PAUSER_ROLE, stakerAddress);
  }
```

