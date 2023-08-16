## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Rebalance will fail due to low precision of percentages](https://github.com/code-423n4/2021-10-union-findings/issues/64) 

# Handle

cmichel


# Vulnerability details

The `AssetManager.rebalance` function has a check at the end to ensure that all tokens are deposited again:

```solidity
require(token.balanceOf(address(this)) == 0, "AssetManager: there are remaining funds in the fund pool");
```

The idea is that the last market deposits all `remainingTokens` but the last market does not have to support the token in which case the transaction will fail, or the `percentages` parameter needs to be chosen to distribute all tokens before the last one (they need to add up to `1e4`). However, these percentages have a low precision as they are in base points, i.e, the lowest unit is `1 = 0.01%`.
This will leave dust in the contract in most cases as the tokens have much higher precision.

## POC
Assume the last market does not support the token and thus `percentages` are chosen as `[5000, 5000]` to rebalance the first two markets.
Withdrawing all tokens form the markets leads to a `tokenSupply = token.balanceOf(address(this)) = 10,001`:

Then the deposited amount is `amountToDeposit = (tokenSupply * percentages[i]) / 10000 = 10,001 * 5,000 / 10,000 = 5,000`.
The two deposits will leave dust of `10,001 - 2 * 5,000 = 1` in the contract and the `token.balanceOf(address(this)) == 0` balance check will revert.

## Impact
Rebalancing will fail in most cases if the last market does not support the token due to precision errors.

## Recommended Mitigation Steps
Remove the final zero balance check, or make sure that the last market that is actually deposited to receives all remaining tokens.


