## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [ecrecover 0 address](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/244) 

# Handle

pauliax


# Vulnerability details

## Impact
ecrecover returns an empty address when the signature is invalid. As far as I checked, with the current codebase, there is no way to exploit it to gain any benefits, but it is a good practice to check against that.
```solidity
  address recoveredAddress = ecrecover(digest, v, r, s);
  require(recoveredAddress == owner, 'YUSD: invalid signature');
```

## Recommended Mitigation Steps
require recoveredAddress != address(0)

You could also consider using OZ's ECDSA library for signature verifications: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol

