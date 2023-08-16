## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Consider making contracts Pausable](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/76) 

# Handle

pauliax


# Vulnerability details

## Impact
There are many external risks (mentioned https://github.com/code-423n4/2021-10-badgerdao#risks) so my suggestion is that you should consider making the contracts pausable, so in case of an unexpected event, the governance can pause transfers.

## Recommended Mitigation Steps
Consider making contracts Pausable https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/Pausable.sol

