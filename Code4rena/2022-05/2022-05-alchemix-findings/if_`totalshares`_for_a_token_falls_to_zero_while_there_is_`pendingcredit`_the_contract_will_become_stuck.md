## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [If `totalShares` for a token falls to zero while there is `pendingCredit` the contract will become stuck](https://github.com/code-423n4/2022-05-alchemix-findings/issues/104) 

# Lines of code

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/AlchemistV2.sol#L1290-L1300
https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/AlchemistV2.sol#L1268
https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/AlchemistV2.sol#L1532
https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/AlchemistV2.sol#L899
https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/AlchemistV2.sol#L1625


# Vulnerability details

## Impact

It is possible for the contract to become stuck and unable to perform any actions if the `totalShares` of a yield token fall to zero while there is some `pendingCredit` still to be paid.

It will then be impossible to call deposit or withdraw functions, mints, burns, repay, liquidate, donate or harvest due to division by zero reverts in:

- `_distributeCredit()`
- `_distributeUnlockedCredit()`
- `_calculateUnrealizedDebt()`
- `_convertSharesToYieldTokens()`
- `donate()`

Furthermore, any `pendingCredit` amount of tokens are still in the contract will become permanently stuck.

## Proof of Concept

This case may arise under the follow steps
a) `deposit()` is called by a user then time passes to earn some yield
b) `harvest()` is called by the keeper which calls `_distributeCredit()` and increases `pendingCredit`
c) `withdraw()` is called by the user to withdraw all funds

Since there is `pendingCredit` the following will have a non-zero balance for `unlockedCredit` however `yieldTokenParams.totalShares` is zero and thus we get a division by zero which reverts the entire transaction.

```solidity
    function _distributeUnlockedCredit(address yieldToken) internal {
        YieldTokenParams storage yieldTokenParams = _yieldTokens[yieldToken];


        uint256 unlockedCredit = _calculateUnlockedCredit(yieldToken);
        if (unlockedCredit == 0) {
            return;
        }


        yieldTokenParams.accruedWeight     += unlockedCredit * FIXED_POINT_SCALAR / yieldTokenParams.totalShares;
        yieldTokenParams.distributedCredit += unlockedCredit;
    }
```

Each of the other listed functions will reach the same issue by attempting to divide some numerator by the `totalShares` which is zero.

## Recommended Mitigation Steps

Consider preventing `totalShares` from over becoming zero once it is set. That is enforce a user to leave at least 1 unit if they are the last user to withdraw.

Another option is to transfer the first 1000 shares to a "burn" account (e.g. 0x000...01), when the first user deposits.

Alternatively, when the last user withdraws, transfer all pending credit to this user and set the required variables to zero to replicate the state before any users have deposited.

