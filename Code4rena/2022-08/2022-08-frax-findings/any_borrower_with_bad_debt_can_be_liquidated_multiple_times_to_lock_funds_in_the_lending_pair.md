## Tags

- bug
- 3 (High Risk)
- high quality report
- sponsor confirmed

# [Any borrower with bad debt can be liquidated multiple times to lock funds in the lending pair](https://github.com/code-423n4/2022-08-frax-findings/issues/102) 

# Lines of code

https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L997-L1015


# Vulnerability details

## Impact

Leftover shares in `liquidateClean` are only subtracted from pair totals, but not from user's borrowed shares. This means that after `liquidateClean`, borrower's shares will be greater than `0` (leftover shares after liquidations), but the user is still insolvent and can be liquidated again and again (with `_sharesToLiquidate` set to `0`). Each subsequent liquidation will write off the bad debt (reduce pair totals by borrower leftover shares/amounts), but doesn't take anything from liquidator nor borrower (since `_sharesToLiquidate == 0`).

This messes up the whole pair accounting, with total asset amounts reducing and total borrow amounts and shares reducing. This will make it impossible for borrowers to repay debt (or be liquidated), because borrow totals will underflow, and lenders amount to withdraw will reduce a lot (they will share non-existant huge bad debt).

Reducing pair totals scenario:
1. Alice borrows `1000 FRAX` (`1000` shares) against `1.5 ETH` collateral (`1 ETH = 1000`, `Max LTV` = `75%`)
2. ETH drops to `500` very quickly with liquidators being unable to liquidate Alice due to network congestion
3. At ETH = `500`, Alice collateral is worth `750` against `1000 FRAX` debt, making Alice insolvent and in a bad debt
4. Liquidator calls `liquidateClean` for `800` shares, which cleans up all available collateral of `1.5 ETH`.
5. At this point Alice has `200` shares debt with `0` collateral
6. Liquidator repeatedly calls `liquidateClean` with `0` shares to liquidate. Each call pair totals are reduced by `200` shares (and total borrow amount by a corresponding amount).
7. When pair totals reach close to `0`, the pool is effectively locked. Borrowers can't repay, lenders can withdraw severly reduced amounts.

## Proof of Concept

Copy this to src/test/e2e/LiquidationBugTest.sol

https://gist.github.com/panprog/cbdc1658d63c30c9fe94127a4b4b7e72


## Recommended Mitigation Steps

After the line

https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L1012

add

        _sharesToLiquidate += _sharesToAdjust;