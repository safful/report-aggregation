## Tags

- bug
- sponsor confirmed
- 1 (Low Risk)

# [Missing overflow protection](https://github.com/code-423n4/2021-11-badgerzaps-findings/issues/13) 

# Handle

pmerkleplant


# Vulnerability details

## Impact
Function `deposit` in `IbbtcVaultZap.sol` computes two additions without
overflow protection, see lines [158](https://github.com/Badger-Finance/badger-ibbtc-utility-zaps/blob/6f700995129182fec81b772f97abab9977b46026/contracts/IbbtcVaultZap.sol#L158) and [166](https://github.com/Badger-Finance/badger-ibbtc-utility-zaps/blob/6f700995129182fec81b772f97abab9977b46026/contracts/IbbtcVaultZap.sol#L166).

In the first case, i.e. line 158, the addition can be changed to an assignment,
as `depositAmount[i]` is always 0.

In the second case, i.e. line 166, an overflow would lead to a wrong amount of
funds deposited into Curve and from there to a wrong amount of LP tokens send
to the `msg.sender`.

## Recommended Steps of Mitigation
As OpenZeppelin's `SafeMathUpgradeable` library is already imported, use
their `add` function instead of the native `+` operator.

