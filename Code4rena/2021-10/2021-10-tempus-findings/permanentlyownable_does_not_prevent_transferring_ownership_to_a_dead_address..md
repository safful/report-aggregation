## Tags

- bug
- sponsor confirmed
- 1 (Low Risk)
- resolved

# [PermanentlyOwnable does not prevent transferring ownership to a dead address.](https://github.com/code-423n4/2021-10-tempus-findings/issues/9) 

# Handle

chenyu


# Vulnerability details

## Impact
[PermanentlyOwnable](https://github.com/code-423n4/2021-10-tempus/blob/main/contracts/utils/PermanentlyOwnable.sol) does not prevent transferring to a dead address. It's possible to have a human error that transfers the contract ownership to a address not owned by the old owner.

## Recommended Mitigation Steps
Recommend a two step transfer that owner nominates an account, then the nominated account call an accept function to ensure the nominated account is valid.

