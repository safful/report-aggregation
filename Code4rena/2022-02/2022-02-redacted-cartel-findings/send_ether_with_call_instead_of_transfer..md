## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Send ether with call instead of transfer.](https://github.com/code-423n4/2022-02-redacted-cartel-findings/issues/2) 

# Lines of code

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L181


# Vulnerability details

## Impact
Use call instead of transfer to send ether. And return value must be checked if sending ether is successful or not.
Sending ether with the transfer is no longer recommended.
## Proof of Concept
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L181

## Tools Used
review
## Recommended Mitigation Steps

(bool result, ) = payable(_account).call{value: _amount}("");
require(result, "Failed to send Ether");

