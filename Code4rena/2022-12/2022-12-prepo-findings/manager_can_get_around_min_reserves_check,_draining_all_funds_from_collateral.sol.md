## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-06

# [Manager can get around min reserves check, draining all funds from Collateral.sol](https://github.com/code-423n4/2022-12-prepo-findings/issues/254) 

# Lines of code

https://github.com/prepo-io/prepo-monorepo/blob/3541bc704ab185a969f300e96e2f744a572a3640/apps/smart-contracts/core/contracts/WithdrawHook.sol#L53-L79


# Vulnerability details

## Impact
When a manager withdraws funds from Collateral.sol, there is a check in the `managerWithdrawHook` to confirm that they aren't pushing the contract below the minimum reserve balance.

```solidity
require(collateral.getReserve() - _amountAfterFee >= getMinReserve(), "reserve would fall below minimum");
```

However, a similar check doesn't happen in the `withdraw()` function. 

The manager can use this flaw to get around the reserve balance by making a large deposit, taking a manager withdrawal, and then withdrawing their deposit.

## Proof of Concept

Imagine a situation where the token has a balance of 100, deposits of 1000, and a reserve percentage of 10%. In this situation, the manager should not be able to make any withdrawal.

But, with the following series of events, they can:
- Manager calls `deposit()` with 100 additional tokens
- Manager calls `managerWithdraw()` to pull 100 tokens from the contract
- Manager calls `withdraw()` to remove the 100 tokens they added

The result is that they are able to drain the balance of the contract all the way to zero, avoiding the intended restrictions.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Include a check on the reserves in the `withdraw()` function as well as `managerWithdraw()`.