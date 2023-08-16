## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Outdated Solidity Version Provides No Protections Against Arithmetic Underflows And Overflows](https://github.com/code-423n4/2021-11-malt-findings/issues/185) 

# Handle

leastwood


# Vulnerability details

## Impact

Malt Finance uses solidity version `>=0.6.6` throughout all of its contracts. This solidity version provides no protections against arithmetic underflows and overflows. As a result, it is incredibly difficult to guarantee that the protocol enforces the necessary arithmetic checks during sensitive actions.

There are several instances where the OpenZeppelin's `SafeMath` library is not used. This exposes the protocol to potential exploits via arithmetic underflows and overflows. The liveness of the protocol depends on safety guarantees that are not provided/enforced. Therefore, this issue should be deemed high severity.

## Proof of Concept

Solidity version shown in all contracts.

## Tools Used

Manual code review.
https://docs.soliditylang.org/en/v0.8.10/080-breaking-changes.html

## Recommended Mitigation Steps

Consider updating the smart contract suite to use the latest solidity version or at the very least integrate OpenZeppelin's `SafeMath` library in all areas of the code containing arithmetic operations.

