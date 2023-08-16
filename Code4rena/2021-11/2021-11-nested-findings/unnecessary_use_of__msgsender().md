## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary Use of _msgSender()](https://github.com/code-423n4/2021-11-nested-findings/issues/185) 

# Handle

defsec


# Vulnerability details

## Impact
The use of _msgSender() when there is no implementation of a meta transaction mechanism that uses it, such as EIP-2771, very slightly increases gas consumption.


## Proof of Concept

1. Navigate to the following contracts.

"https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L135"
"https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedFactory.sol#L111"
"https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedReserve.sol#L32"

## Tools Used

Code review

## Recommended Mitigation Steps

Replace _msgSender() with msg.sender if there is no mechanism to support meta-transactions like EIP-2771 implemented.



