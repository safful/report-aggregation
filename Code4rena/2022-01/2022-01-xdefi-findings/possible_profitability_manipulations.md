## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Possible profitability manipulations](https://github.com/code-423n4/2022-01-xdefi-findings/issues/193) 

# Handle

Czar102


# Vulnerability details

## Impact

An owner of the contract may, by ordering their or others' locking transactions, significantly increase or decrease `bonusMultiplier` for some set of transactions.

So, by mining a single block, an owner can lower other's bonus multipliers, execute locking transactions and then restore bonus multipliers.

A user might send a locking transaction in a similar time as an owner lowers the multipliers, resulting in lowering the revenue against data presented to the user.

An owner can also pass ownership to a contract that will change bonus multipliers and lock funds with a very high bonus multiplier, then restore previous multipliers' state not to let others do the same. This way, the owner can gain an unfair advantage over others.

## Recommended Mitigation Steps

Use a timelock for `setLockPeriods(...)` function and require passing `bonusMultiplier` in locking functions, revert if they are different from the state variables.


