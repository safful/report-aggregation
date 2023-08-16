## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Consider making some constants as non-public to save gas](https://github.com/code-423n4/2022-01-livepeer-findings/issues/63) 

# Handle

Dravee


# Vulnerability details

## Impact  
Reducing from public to private will save gas
  
## Proof of Concept  
```
arbitrum-lpt-bridge\contracts\L1\gateway\L1Migrator.sol:111:    bytes32 public constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
arbitrum-lpt-bridge\contracts\L2\gateway\L2Migrator.sol:59:    bytes32 public constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
arbitrum-lpt-bridge\contracts\L2\token\LivepeerToken.sol:9:    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
arbitrum-lpt-bridge\contracts\L2\token\LivepeerToken.sol:10:    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
arbitrum-lpt-bridge\contracts\ControlledGateway.sol:13:    bytes32 public constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
``` 
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Theses constants can simply be read from the verified contract, i.e., it is unnecessary to expose them with a public function.

