## Tags

- bug
- duplicate
- 0 (Non-critical)
- sponsor confirmed

# [lack of zero address validation in constructor](https://github.com/code-423n4/2021-08-notional-findings/issues/51) 

# Handle

JMukesh


# Vulnerability details

## Impact
since the parameter of the constructor are used to initialize the sate variable and these state variable are used throughout the contract error in these parameter can lead to redeployment of the contract

## Proof of Concept

constructor of ctokenAggregator.sol,  NotionalV1ToNotionalV2.sol, nTokenERC20Proxy.sol, Reservoir.sol, PauseRouter.sol lack zero address validation

## Tools Used
manual review

## Recommended Mitigation Steps
add address(0) validation in constructor

