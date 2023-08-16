## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# ['onlyAdmin' and 'onlySwivel' modifiers](https://github.com/code-423n4/2021-09-swivel-findings/issues/148) 

# Handle

pauliax


# Vulnerability details

## Impact
Having both modifiers 'onlyAdmin' and 'onlySwivel' is not only more expensive but also misleading as these modifiers basically do the same job of checking an address against msg.sender.

## Recommended Mitigation Steps
Better have a generalized modifier, something like onlyAddress(address a), and re-use it with both admin and swivel:
  modifier onlyAddress(address a) {
    require(msg.sender == a, 'sender not authorized');
    _;
  }
  onlyAddress(admin)
  onlyAddress(swivel)

