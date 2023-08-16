## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: use `msg.sender` instead of OpenZeppelin's `_msgSender()` when GSN capabilities aren't used](https://github.com/code-423n4/2022-01-livepeer-findings/issues/243) 

# Handle

Dravee


# Vulnerability details

## Impact
`msg.sender` costs 2 gas (CALLER opcode).
`_msgSender()` represents the following:
```
function _msgSender() internal view virtual returns (address payable) {
    return msg.sender;
}
```
When no GSN capabilities are used: `msg.sender` is enough.

See https://docs.openzeppelin.com/contracts/2.x/gsn for more information about GSN capabilities.

## Proof of Concept
Instances include:
```
arbitrum-lpt-bridge\contracts\L1\escrow\L1Escrow.sol:18:        _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());    
arbitrum-lpt-bridge\contracts\L1\gateway\L1Migrator.sol:133:        _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());
arbitrum-lpt-bridge\contracts\L2\gateway\L2Migrator.sol:83:        _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());
arbitrum-lpt-bridge\contracts\L2\token\LivepeerToken.sol:13:        _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());
arbitrum-lpt-bridge\contracts\ControlledGateway.sol:19:        _setupRole(DEFAULT_ADMIN_ROLE, _msgSender());
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Replace `_msgSender()` with `msg.sender`

