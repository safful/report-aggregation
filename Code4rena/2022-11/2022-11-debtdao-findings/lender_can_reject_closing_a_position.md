## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- M-11

# [Lender can reject closing a position](https://github.com/code-423n4/2022-11-debtdao-findings/issues/467) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L489-L493


# Vulnerability details

A credit line can be closed by using the `LineOfCredit.depositAndClose()` or `LineOfCredit.close`. The remaining funds deposited by the lender (`credit.deposit`) and the accumulated and paid interest are transferred to the lender.

However, if the used credit token `credit.token` is native ETH (or an ERC-777 token with receiver hooks, and under the assumption that the oracle supports this asset in the first place), the lender can reject the closing of the credit by reverting the token transfer.

## Impact

The lender can prevent the borrower from closing the credit line. This leads to the following consequences:

- Migrating (rollover) to a new line is not possible (it requires all credits to be closed, see [SecuredLine.sol#L55](https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/SecuredLine.sol#L55))
- Releasing a spigot and transferring ownership to the borrower is not possible (see [SpigotedLineLib.sol#L195](https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/SpigotedLineLib.sol#L195))
- Sweeping remaining tokens (e.g. revenue tokens) in the Spigot to the borrower is not possible (see (SpigotedLineLib.sol#L220)[https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/SpigotedLineLib.sol#L220])

## Proof of Concept

[modules/credit/LineOfCredit.sol#L489-L493](https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L489-L493)

```solidity
function _close(Credit memory credit, bytes32 id) internal virtual returns (bool) {
    if(credit.principal > 0) { revert CloseFailedWithPrincipal(); }

    // return the Lender's funds that are being repaid
    if (credit.deposit + credit.interestRepaid > 0) {
        LineLib.sendOutTokenOrETH(
            credit.token,
            credit.lender,
            credit.deposit + credit.interestRepaid
        );
    }

    delete credits[id]; // gas refunds

    // remove from active list
    ids.removePosition(id);
    unchecked { --count; }

    // If all credit lines are closed the the overall Line of Credit facility is declared 'repaid'.
    if (count == 0) { _updateStatus(LineLib.STATUS.REPAID); }

    emit CloseCreditPosition(id);

    return true;
}
```

## Tools Used

Manual review

## Recommended mitigation steps

Consider using a pull-based pattern to allow the lender to withdraw the funds instead of sending them back directly.
