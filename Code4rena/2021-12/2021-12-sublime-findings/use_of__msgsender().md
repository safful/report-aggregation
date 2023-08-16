## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use of _msgSender()](https://github.com/code-423n4/2021-12-sublime-findings/issues/102) 

# Handle

defsec


# Vulnerability details

## Impact

The use of _msgSender() when there is no implementation of a meta transaction mechanism that uses it, such as EIP-2771, very slightly increases gas consumption.


## Proof of Concept

_msgSender() is utilized three times where msg.sender could have been used in the following function.


"""
https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/yield/YearnYield.sol#L42

https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/yield/CompoundYield.sol#L43

https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/yield/AaveYield.sol#L71

https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/yield/NoYield.sol#L31

"""


## Tools Used

None

## Recommended Mitigation Steps

Replace _msgSender() with msg.sender if there is no mechanism to support meta-transactions like EIP-2771 implemented.

