## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Named Return Issues](https://github.com/code-423n4/2021-10-tempus-findings/issues/4) 

# Handle

ye0lde


# Vulnerability details

## Impact

Removing unused named return variables can reduce gas usage and improve code clarity.

## Proof of Concept

AaveTempusPool.sol:
Unused named return
https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/pools/AaveTempusPool.sol#L74

LidoTempusPool.sol:
Unused named return
https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/pools/LidoTempusPool.sol#L59

TempusAMM.sol:
Unneeded return
https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/amm/TempusAMM.sol#L533

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Remove the unused named return variables or return.

