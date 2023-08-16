## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- BadgerYieldSource

# [BadgerYieldSource balanceOfToken share calculation seems wrong](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/84) 

# Handle

cmichel


# Vulnerability details


When suppling to the `BadgerYieldSource`, some `amount` of `badger` is deposited to `badgerSett` and one receives `badgerSett` share tokens in return which are stored in the `balances` mapping of the user. So far this is correct.

The `balanceOfToken` function should then return the redeemable balance in `badger` for the user's `badgerSett` balance.
It computes it as the pro-rata share of the user balance (compared to the total-supply of `badgerSett`) on the `badger` in the vault:

```solidity
balances[addr].mul(
  badger.balanceOf(address(badgerSett))
).div(
  badgerSett.totalSupply()
)
```

However, `badger.balanceOf(address(badgerSett))` is only a small amount of badger that is deployed in the vault ("Sett") due to most of the capital being deployed to the _strategies_. Therefore, it under-reports the actual balance:

> Typically, a Sett will keep a small portion of deposited funds in reserve to handle small withdrawals cheaply. [Badger Docs](https://badger-finance.gitbook.io/badger-finance/technical/setts/sett-contract)

## Impact

Any contract or user calling the `balanceOf` function will receive a value that is far lower than the actual balance.
Using this value as a basis for computations will lead to further errors in the integrations.

## Recommended Mitigation Steps

It should use [`badgerSett.balance()`](https://github.com/Badger-Finance/badger-system/blob/2b0ee9bd77a2cc6f875b9b984ae4dfe713bbc55c/contracts/badger-sett/Sett.sol#L126) instead of `badger.balanceOf(address(badgerSett))` to also account for "the balance in the Sett, the Controller, and the Strategy".

