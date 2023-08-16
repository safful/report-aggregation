## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Custom GOVERNOR_ROLE unnecessary](https://github.com/code-423n4/2022-01-livepeer-findings/issues/116) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The ControlledGateway.sol contract specifies a custom "GOVERNOR_ROLE" value that is assigned to the _msgsender when the contract is deployed. There is no need to create a custom role when only one role is used in the contract. This custom "GOVERNOR_ROLE" could be replaced with the built-in "DEFAULT_ADMIN_ROLE" value, which is the approach in the contract L1/escrow/L1Escrow.sol.

## Proof of Concept

The custom role is created [on line 13 of ControlledGateway.sol](https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/ControlledGateway.sol#L13)

## Recommended Mitigation Steps

Remove the GOVERNOR_ROLE role in ControlledGateway.sol and use the built-in DEFAULT_ADMIN_ROLE role to save gas

