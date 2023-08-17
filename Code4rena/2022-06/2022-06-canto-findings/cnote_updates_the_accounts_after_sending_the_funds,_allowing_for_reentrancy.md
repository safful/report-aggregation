## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [CNote updates the accounts after sending the funds, allowing for reentrancy](https://github.com/code-423n4/2022-06-canto-findings/issues/311) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/2d423c7c3f62d65182d802deb99cc7bba4e057fd/contracts/CNote.sol#L70-L87


# Vulnerability details

Having no reentrancy control and updating the records after external interactions allows for funds draining by reentrancy.

Setting the severity to medium as this is conditional to transfer flow control introduction on future upgrades, but the impact is up to the full loss of the available funds by unrestricted borrowing.

## Proof of Concept

CNote runs doTransferOut before borrowing accounts are updated:

https://github.com/Plex-Engineer/lending-market/blob/2d423c7c3f62d65182d802deb99cc7bba4e057fd/contracts/CNote.sol#L70-L87

```
        /*
         * We invoke doTransferOut for the borrower and the borrowAmount.
         *  Note: The cToken must handle variations between ERC-20 and ETH underlying.
         *  On success, the cToken borrowAmount less of cash.
         *  doTransferOut reverts if anything goes wrong, since we can't be sure if side effects occurred.
         */
        doTransferOut(borrower, borrowAmount);
        require(getCashPrior() == 0,"CNote::borrowFresh: Error in doTransferOut, impossible Liquidity in LendingMarket");
    //Amount minted by Accountant is always flashed from account
    
    /* We write the previously calculated values into storage */
        accountBorrows[borrower].principal = accountBorrowsNew;
        accountBorrows[borrower].interestIndex = borrowIndex;
        totalBorrows = totalBorrowsNew;

        /* We emit a Borrow event */
        emit Borrow(borrower, borrowAmount, accountBorrowsNew, totalBorrowsNew);
    }
```

Call sequence here is borrow() -> borrowInternal() -> borrowFresh() -> doTransferOut(), which transfers the token to an external recipient:

https://github.com/Plex-Engineer/lending-market/blob/2d423c7c3f62d65182d802deb99cc7bba4e057fd/contracts/CErc20.sol#L189-L200

```
    /**
     * @dev Similar to EIP20 transfer, except it handles a False success from `transfer` and returns an explanatory
     *      error code rather than reverting. If caller has not called checked protocol's balance, this may revert due to
     *      insufficient cash held in this contract. If caller has checked protocol's balance prior to this call, and verified
     *      it is >= amount, this should not revert in normal conditions.
     *
     *      Note: This wrapper safely handles non-standard ERC-20 tokens that do not return a value.
     *            See here: https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca
     */
    function doTransferOut(address payable to, uint amount) virtual override internal {
        EIP20NonStandardInterface token = EIP20NonStandardInterface(underlying);
        token.transfer(to, amount);
```

There an attacker can call exitMarket() that have no reentrancy control to remove the account of the debt:

https://github.com/Plex-Engineer/lending-market/blob/2d423c7c3f62d65182d802deb99cc7bba4e057fd/contracts/Comptroller.sol#L167-L174

https://github.com/Plex-Engineer/lending-market/blob/2d423c7c3f62d65182d802deb99cc7bba4e057fd/contracts/ComptrollerG7.sol#L157-L164

```
    /**
     * @notice Removes asset from sender's account liquidity calculation
     * @dev Sender must not have an outstanding borrow balance in the asset,
     *  or be providing necessary collateral for an outstanding borrow.
     * @param cTokenAddress The address of the asset to be removed
     * @return Whether or not the account successfully exited the market
     */
    function exitMarket(address cTokenAddress) override external returns (uint) {
```

This attack was carried out several times:

https://certik.medium.com/fei-protocol-incident-analysis-8527440696cc


## Recommended Mitigation Steps

Consider moving accounting update before funds were sent out, for example as it is done in CToken's borrowFresh():

```
https://github.com/Plex-Engineer/lending-market/blob/2d423c7c3f62d65182d802deb99cc7bba4e057fd/contracts/CToken.sol#L595-L609

        /*
         * We write the previously calculated values into storage.
         *  Note: Avoid token reentrancy attacks by writing increased borrow before external transfer.
        `*/
        accountBorrows[borrower].principal = accountBorrowsNew;
        accountBorrows[borrower].interestIndex = borrowIndex;
        totalBorrows = totalBorrowsNew;

        /*
         * We invoke doTransferOut for the borrower and the borrowAmount.
         *  Note: The cToken must handle variations between ERC-20 and ETH underlying.
         *  On success, the cToken borrowAmount less of cash.
         *  doTransferOut reverts if anything goes wrong, since we can't be sure if side effects occurred.
         */
        doTransferOut(borrower, borrowAmount);
```

