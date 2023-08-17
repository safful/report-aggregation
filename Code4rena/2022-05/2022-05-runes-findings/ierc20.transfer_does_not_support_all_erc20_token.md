## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [IERC20.transfer does not support all ERC20 token](https://github.com/code-423n4/2022-05-runes-findings/issues/70) 

# Lines of code

https://github.com/code-423n4/2022-05-runes/blob/060b4f82b79c8308fe65674a39a07c44fa586cd3/contracts/ForgottenRunesWarriorsGuild.sol#L173-L176
https://github.com/code-423n4/2022-05-runes/blob/060b4f82b79c8308fe65674a39a07c44fa586cd3/contracts/ForgottenRunesWarriorsMinter.sol#L627-L630


# Vulnerability details

Token like [USDT](https://etherscan.io/address/0xdac17f958d2ee523a2206206994597c13d831ec7#contracts) known for using non-standard ERC20. ([Missing return boolean on transfer](https://forum.openzeppelin.com/t/can-not-call-the-function-approve-of-the-usdt-contract/2130/4)).

Contract function [forwardERC20](https://github.com/code-423n4/2022-05-runes/blob/060b4f82b79c8308fe65674a39a07c44fa586cd3/contracts/ForgottenRunesWarriorsGuild.sol#L173-L176) will always revert when try to transfer this kind of tokens.

## Impact

Cannot withdraw some special ERC20 token through contract call. Unexpected contract functionality = Medium severity

## Migration

Use [SafeTransferLib.safeTransfer](https://github.com/Rari-Capital/solmate/blob/4197b521ef3eb81f675d35e64b7b597b24d33500/src/utils/SafeTransferLib.sol#L65-L94) instead of IERC20 transfer. This accepts ERC20 token with no boolean return like USDT.


