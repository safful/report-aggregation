## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Guard for initialization function of VaultGovernance](https://github.com/code-423n4/2021-12-mellow-findings/issues/105) 

# Handle

cuong_qnom


# Vulnerability details

### Impact
There should be a guard to initialize the factory in the VaultGovernance. Otherwise, some guys (e.g. miners) can front-run the initialization transaction with fake Factory address. 
### Proof of Concept
https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/VaultGovernance.sol#L77
### Tools used
Manual Analysis
### Recommendation steps
Maybe can put some requirements at the start:
_requireProtocolAdmin()


