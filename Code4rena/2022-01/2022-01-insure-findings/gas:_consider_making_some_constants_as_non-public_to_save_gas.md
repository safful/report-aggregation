## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Consider making some constants as non-public to save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/30) 

# Handle

Dravee


# Vulnerability details

## Impact  
Reducing from public to private will save gas
  
## Proof of Concept  
```  
PremiumModels\BondingPremium.sol:26:    //constants
PremiumModels\BondingPremium.sol:27:    uint256 public constant DECIMAL = uint256(1e6); //Decimals of USDC
PremiumModels\BondingPremium.sol:28:    uint256 public constant BASE = uint256(1e6); //bonding curve graph takes 1e6 as 100.0000%
PremiumModels\BondingPremium.sol:29:    uint256 public constant BASE_x2 = uint256(1e12); //BASE^2
PremiumModels\BondingPremium.sol:30:    uint256 public constant ADJUSTER = uint256(10); //adjuster of 1e6 to 1e5 (100.0000% to 100.000%)
CDSTemplate.sol:55:    uint256 public constant MAGIC_SCALE_1E6 = 1e6; //internal multiplication scale 1e6 to reduce decimal truncation
IndexTemplate.sol:95:    uint256 public constant MAGIC_SCALE_1E6 = 1e6; //internal multiplication scale 1e6 to reduce decimal truncation
PoolTemplate.sol:146:    uint256 public constant MAGIC_SCALE_1E6 = 1e6; //internal multiplication scale 1e6 to reduce decimal truncation
Vault.sol:38:    uint256 public constant MAGIC_SCALE_1E6 = 1e6; //internal multiplication scale 1e6 to reduce decimal truncation
```  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Theses constants can simply be read from the verified contract, i.e., it is unnecessary to expose it with a public function.
Also, constants having "1E6" in their name aren't even "nice to have public constants", as their value is obvious.


