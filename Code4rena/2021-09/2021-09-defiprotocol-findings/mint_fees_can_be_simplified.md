## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Mint fees can be simplified](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/214) 

# Handle

pauliax


# Vulnerability details

## Impact
This can be refactored to improve precision and gas usage:
  _mint(publisher, fee * (BASE - factory.ownerSplit()) / BASE);
  _mint(Ownable(address(factory)).owner(), fee * factory.ownerSplit() / BASE);

## Recommended Mitigation Steps
Proposed solution:
  uint256 factoryOwnerFee = fee * factory.ownerSplit() / BASE;
  uint256 publisherFee = fee - factoryOwnerFee;
  _mint(Ownable(address(factory)).owner(), factoryOwnerFee);
  _mint(publisher, publisherFee);
This will result in fewer math operations and better precision cuz multiplication and division are replaced with subtraction.

