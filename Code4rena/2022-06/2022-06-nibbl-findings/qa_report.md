## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-06-nibbl-findings/issues/255) 

# QA Report

## [L-01] Lack of ownership transfer pattern
Setting the curator to the wrong address will result in permanent loss of functionality in the protocol. It is recommended to apply a two-step transfer.

https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVault.sol#L485-L488

## [L-02] Unnecessary safeTransferFrom() when sending from address(this)
`safeTransfer()` can be used instead when sending from the current address.

https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVault.sol#L495-L509

## [L-03] Missing zero address check
Contract would need to be redeployed if the `feeTo` variable is accidently set to address(0).

https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVaultFactory.sol#L25

## [L-04] Unsafe transfer, use safeTransfer()
It is recommended to use safeTransfer() instead of transfer() to mitigate against ERC20 implementations that do not revert on failure.

https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVault.sol#L517

## [N-01] Constants should be all caps
Most constants defined in the contracts are fully capitalized, just this one example is not.

https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVault.sol#L28

## [N-02] CURVE_FEE is described as a variable when it is a constant
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVault.sol#L210