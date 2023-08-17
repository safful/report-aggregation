## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`VE3DRewardPool` claim in loop depend on pausable token](https://github.com/code-423n4/2022-05-vetoken-findings/issues/166) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VE3DRewardPool.sol#L296-L299
https://github.com/code-423n4/2022-05-vetoken/blob/1be2f03670e407908f175c08cf8cc0ce96c55baf/contracts/VeAssetDepositor.sol#L134-L152


# Vulnerability details

Project veToken is supposed to be a generalized version of Convex for non-Curve token.
There is only one contract for all rewards token in the platform.

All ve3Token rewards are bundled together inside `ve3DLocker` and `ve3DRewardPool` in a loop. Instead of having its own unique contract like `VeAssetDepositer` or `VoterProxy` for each token.

## Impact

If one token has pausable transfer, user cannot claim rewards or withdraw if they have multiple rewards include that pause token.

Right now the project intends to support only 6 tokens, including Ribbon token which has [pausable transfer](https://etherscan.io/address/0x6123b0049f904d730db3c36a31167d9d4121fa6b#code#L810) controlled by Ribbon DAO.

Normally, this would not be an issue in Convex where only a few pools would be affected by single coin. Since, veAsset are bundled together into single reward pool, it becomes a major problem.

## Proof of concept

- Token like Ribbon pause token transfer by DAO due to an unfortunate event.
- `VE3DRewardPool` try call `getReward()`, `VeAssetDepositor` [try deposit token from earned rewards](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VE3DRewardPool.sol#L296-L299) does not work anymore because `IERC20.transfer` [is blocked](https://github.com/code-423n4/2022-05-vetoken/blob/1be2f03670e407908f175c08cf8cc0ce96c55baf/contracts/VeAssetDepositor.sol#L134-L152). This effectively reverts current function if user have this token reward > 0.

## Recommended mitigation step

It would be a better practice if we had a second `getReward()` function that accepts an array of token that we would like to interact with.
It saves gas and only requires some extra work on frontend website.
Instead of current implementation, withdraw all token bundles together.


