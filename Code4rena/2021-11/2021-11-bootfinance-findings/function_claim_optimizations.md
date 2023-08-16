## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [function claim optimizations](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/283) 

# Handle

pauliax


# Vulnerability details

## Impact
function claim can save gas and eliminate duplicate storage access and math operations by caching claimableAmount and re-using it later when setting the benClaimed. 

before:
  uint256 amount = _claimableAmount(msg.sender).sub(benClaimed[msg.sender]);
  ...
  benClaimed[msg.sender] = benClaimed[msg.sender].add(amount);

after:
  uint256 claimableAmount = _claimableAmount(msg.sender);
  uint256 amount = claimableAmount.sub(benClaimed[msg.sender]);
  ...
  benClaimed[msg.sender] = claimableAmount;

Also, it looks strange that in function revoke the amount is checked with 'assert':
assert(amount <= benTotal[_addr]);
but in function claim 'require' is used:
require(amount <= benTotal[msg.sender], "Cannot withdraw more than total vested amount");

In both places probably 'assert' should be used as it is checking a scenario that should never happen under normal circumstances.


