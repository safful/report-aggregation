## Tags

- bug
- QA (Quality Assurance)
- resolved
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-04-jpegd-findings/issues/90) 

1. It was found some `transfer` or `transferFrom` without checking the boolean result, ERC20 standard specify that the token can return false if this call was not made, so it's mandatory to check the result of approve methods.
- [NFTVault.sol#L899](https://github.com/code-423n4/2022-04-jpegd/blob/e72861a9ccb707ced9015166fbded5c97c6991b6/contracts/vaults/NFTVault.sol#L899)
- [JPEGStaking.sol#L34](https://github.com/code-423n4/2022-04-jpegd/blob/e72861a9ccb707ced9015166fbded5c97c6991b6/contracts/staking/JPEGStaking.sol#L34)
- [JPEGStaking.sol#L52](https://github.com/code-423n4/2022-04-jpegd/blob/e72861a9ccb707ced9015166fbded5c97c6991b6/contracts/staking/JPEGStaking.sol#L52)

2. Lack of input checks.
- [LPFarming.sol#L77](https://github.com/code-423n4/2022-04-jpegd/blob/e72861a9ccb707ced9015166fbded5c97c6991b6/contracts/farming/LPFarming.sol#L77)
