## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Incorrect deployment parameters](https://github.com/code-423n4/2022-05-vetoken-findings/issues/52) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/migrations/25_deploy_angle_pools.js#L68
https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/migrations/25_deploy_angle_pools.js#L80


# Vulnerability details

## Impact
The address of G-Uni tokens in the deployment scripts are not up to date. 

## Proof of concept
For example for agEUR/USDC it is 0xedecb43233549c51cc3268b5de840239787ad56c and not 0x2bD9F7974Bc0E4Cb19B8813F8Be6034F3E772add

## Mitigation steps
For safety why not fetching directly the LP token from the staking contract ? 

