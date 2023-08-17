## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Div by 0](https://github.com/code-423n4/2022-03-volt-findings/issues/58) 

# Lines of code

https://github.com/code-423n4/2022-03-volt/tree/main/contracts/utils/Deviation.sol#L23


# Vulnerability details


Division by 0 can lead to accidentally revert,
(An example of a similar issue - https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/84)

        https://github.com/code-423n4/2022-03-volt/tree/main/contracts/utils/Deviation.sol#L23 a might be 0

It's internal function but since it is used in another internal functions that are used in public and neither of them has this protection I thought it can be considered as medium (e.g. isWithinDeviationThreshold)

Thanks.

