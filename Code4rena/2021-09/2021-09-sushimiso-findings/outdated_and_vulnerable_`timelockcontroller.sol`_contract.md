## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [Outdated and Vulnerable `TimelockController.sol` Contract](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/24) 

# Handle

leastwood


# Vulnerability details

## Impact

`TimelockController.sol` acts as an auxiliary contract to the MISO platform's core contracts. Therefore, this issue is not of high risk as not all users wanting to auction tokens will use this contract for governance behaviour. The `TimelockController.sol` enables a governance framework to enforce a timelock on any proposals, giving users time to exit before a potentially dangerous maintenance operation is applied. However, the `executeBatch()` is vulnerable to reentrancy, enabling privilege escalation for any account with the `EXECUTOR` role to `ADMIN`.

## Proof of Concept

Bug outlined [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.4.0-solc-0.7/contracts/access/TimelockController.sol#L244-L269).
Fix is outlined in this [commit](https://github.com/OpenZeppelin/openzeppelin-contracts/commit/cec4f2ef57495d8b1742d62846da212515d99dd5#diff-8229f9027848871a1706845a5a84fa3e6591445cfac6e16cfb7d652e91e8d395R307).

## Tools Used

Sourced from publicly disclosed post by [Immnuefi](https://medium.com/immunefi/openzeppelin-bug-fix-postmortem-66d8c89ed166).

## Recommended Mitigation Steps

Update `Openzeppelin` library to a version containing the commit fixing the bug (mentioned above). Tag `v3.4.2-solc-0.7` in `Openzeppelin`'s Github repository is an example of a compatible library that contains the aforementioned bug fix.

