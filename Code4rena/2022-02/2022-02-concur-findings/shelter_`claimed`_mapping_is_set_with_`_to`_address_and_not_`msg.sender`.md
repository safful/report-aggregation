## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Shelter `claimed` mapping is set with `_to` address and not `msg.sender`](https://github.com/code-423n4/2022-02-concur-findings/issues/103) 

# Lines of code

 https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/Shelter.sol#L55


# Vulnerability details

# Impact

Any user can withdraw all the funds from the shelter. This is done by calling withdraw repeatedly until all funds are drained. You only need to have a small share.

Even if the `claimed` mapping was checked, there would still be a vulnerability. This is because the `claimed` mapping is updated with the `_to` address, not the `msg.sender` address.

Remediation is to change the `_to` to `msg.sender`.
https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/Shelter.sol#L55


