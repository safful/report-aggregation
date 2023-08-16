## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Deprecated safeApprove() function](https://github.com/code-423n4/2021-12-sublime-findings/issues/2) 

# Handle

sirhashalot


# Vulnerability details

The OpenZeppelin ERC20 `safeApprove()` function has been deprecated, as seen in the comments of the OpenZeppelin code.

## Impact
Detailed description of the impact of this finding.

Using this deprecated function can lead to unintended reverts and potentially the locking of funds. A deeper discussion on the deprecation of this function is in OZ [issue #2219](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2219).

## Proof of Concept

The deprecated function is found in:
- SavingsAccount/SavingsAccount.sol [line 173](https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/SavingsAccount/SavingsAccount.sol#L173)
- SavingsAccount/SavingsAccountUtil.sol [line 61](https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/SavingsAccount/SavingsAccountUtil.sol#L61)
- mocks/yVault/yVault.sol [line 164](https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/mocks/yVault/yVault.sol#L164)
- mocks/yVault/Controller.sol [line 196 and 197](https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/mocks/yVault/Controller.sol#L196)


## Tools Used

Manual analysis

## Recommended Mitigation Steps

As suggested by the OpenZeppelin comment, replace `safeApprove()` with `safeIncreaseAllowance()` or `safeDecreaseAllowance()` instead.

