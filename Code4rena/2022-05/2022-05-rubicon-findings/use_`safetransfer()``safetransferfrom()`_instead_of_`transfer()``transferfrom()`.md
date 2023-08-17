## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Use `safeTransfer()`/`safeTransferFrom()` instead of `transfer()`/`transferFrom()`](https://github.com/code-423n4/2022-05-rubicon-findings/issues/316) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L251


# Vulnerability details

## Impact

It is a good idea to add a `require()` statement that checks the return value of ERC20 token transfers or to use something like OpenZeppelin’s `safeTransfer()`/`safeTransferFrom()` unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

However, using `require()` to check transfer return values could lead to issues with non-compliant ERC20 tokens which do not return a boolean value. Therefore, it's highly advised to use OpenZeppelin’s `safeTransfer()`/`safeTransferFrom()`.

## Proof of Concept

**RubiconRouter.sol**

[L251](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L251): `ERC20(route[route.length - 1]).transfer(to, currentAmount);`\
[L303](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L303): `ERC20(buy_gem).transfer(msg.sender, fill);`\
[L320](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L320): `ERC20(buy_gem).transfer(msg.sender, fill);`\
[L348](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L348): `ERC20(buy_gem).transfer(msg.sender, buy_amt);`\
[L377](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L377): `ERC20(pay_gem).transfer(msg.sender, max_fill_amount - fill);`\
[L406](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L406): `ERC20(buy_gem).transfer(msg.sender, _after - _before);`\
[L471](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L471): `ERC20(targetPool).transfer(msg.sender, newShares);`

**peripheral_contracts/BathBuddy.sol**

[L114](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L114): `token.transfer(recipient, amountWithdrawn);`

**rubiconPools/BathPair.sol**

[L601](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathPair.sol#L601): `IERC20(asset).transfer(msg.sender, booty);`\
[L615](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathPair.sol#L615): `IERC20(quote).transfer(msg.sender, booty);`

**rubiconPools/BathToken.sol**

[L353](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L353): `IERC20(filledAssetToRebalance).transfer(`\
[L357](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L357): `IERC20(filledAssetToRebalance).transfer(msg.sender, stratReward); `\
[L602](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L602): `underlyingToken.transfer(feeTo, _fee);`\
[L605](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L605): `underlyingToken.transfer(receiver, amountWithdrawn);`

## Tools Used

Manual review

## Recommended mitigation steps

Consider using `safeTransfer()`/`safeTransferFrom()` instead of `transfer()`/`transferFrom()`.

