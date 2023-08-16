## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Constant variables using keccak can be immutable](https://github.com/code-423n4/2022-01-livepeer-findings/issues/172) 

# Handle

ye0lde


# Vulnerability details

## Impact
Changing the variables from constant to immutable will reduce keccak operations and save gas.

A previous finding with additional explanation and a pointer to the ethereum/solidity issue is here:
https://github.com/code-423n4/2021-10-slingshot-findings/issues/3

## Proof of Concept
These variables can simply be changed from `constant` to `immutable`:
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L114
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L116
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L121

Additional changes are needed for these variables since they are used in the constructor:
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L111
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2Migrator.sol#L59
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/token/LivepeerToken.sol#L9-L10
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/ControlledGateway.sol#L13


Here's an example of the changes needed in the constructor for:
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/ControlledGateway.sol#L13

```
contract ControlledGateway is AccessControl, Pausable {
    bytes32 public immutable GOVERNOR_ROLE;  

    address public immutable l1Lpt;
    address public immutable l2Lpt;

    constructor(address _l1Lpt, address _l2Lpt) {
        _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());
        _setRoleAdmin(GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE"), DEFAULT_ADMIN_ROLE);

        l1Lpt = _l1Lpt;
        l2Lpt = _l2Lpt;
    }

    function pause() external onlyRole(GOVERNOR_ROLE) {
        _pause();
    }

    function unpause() external onlyRole(GOVERNOR_ROLE) {
        _unpause();
    }
}
```

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Change the constant variables to immutable as described in the POC.



