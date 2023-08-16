## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [safeERC20 library imported but not used](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/154) 

# Handle

loop


# Vulnerability details

`AirDropDistribution.sol` and `InvestorDistribution.sol` import the `safeERC20` library but make use of the normal ERC20 `transfer` function rather than `safeTransfer`. Considering this is called on the BOOT token there is likely no need for it to be `safeTransfer`. However, since the library is not used there is no need for it to be imported. 

## Proof of Concept
- https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L12
- https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L12

Transfer calls:
- https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L542
- https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L567

- https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L132
- https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L156
- https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L207

