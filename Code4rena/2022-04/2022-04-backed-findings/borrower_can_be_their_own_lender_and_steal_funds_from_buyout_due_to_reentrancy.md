## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Borrower can be their own lender and steal funds from buyout due to reentrancy](https://github.com/code-423n4/2022-04-backed-findings/issues/85) 

# Lines of code

https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L214-L221
https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L230-L250


# Vulnerability details

## Impact
If borrower lends their own loan, they can repay and close the loan before ownership of the lend ticket is transferred to the new lender. The borrower will keep the NFT + loan amount + accrued interest.

## Proof of Concept
This exploit requires that the `loanAssetContractAddress` token transfers control to the receiver.

### Steps of exploit:

- Borrower creates loan with `createLoan()`.
- The same Borrower calls `lend()`, funding their own loan. The Borrower receives the lend ticket, and funds are transferred to themself.
- A new lender attempts to buy out the loan. The original loan amount + accruedInterest are sent to the original lender (same person as borrower).
- Due to lack of checks-effects-interactions pattern, the borrower is able to immediately call `repayAndCloseLoan()` before the lend ticket is transferred to the new lender.

The following code illustrates that the new lender sends funds to the original lender prior to receiving the lend ticket in return.

```
            } else {
                ERC20(loan.loanAssetContractAddress).safeTransferFrom(
                    msg.sender,
                    currentLoanOwner,
                    accumulatedInterest + previousLoanAmount
                );
            }
            ILendTicket(lendTicketContract).loanFacilitatorTransfer(currentLoanOwner, sendLendTicketTo, loanId);
```

The original lender/borrower calls the following `repayAndCloseLoan()` function so that they receive their collateral NFT from the protocol.

```
    function repayAndCloseLoan(uint256 loanId) external override notClosed(loanId) {
        Loan storage loan = loanInfo[loanId];


        uint256 interest = _interestOwed(
            loan.loanAmount,
            loan.lastAccumulatedTimestamp,
            loan.perAnumInterestRate,
            loan.accumulatedInterest
        );
        address lender = IERC721(lendTicketContract).ownerOf(loanId);
        loan.closed = true;
        ERC20(loan.loanAssetContractAddress).safeTransferFrom(msg.sender, lender, interest + loan.loanAmount);
        IERC721(loan.collateralContractAddress).safeTransferFrom(
            address(this),
            IERC721(borrowTicketContract).ownerOf(loanId),
            loan.collateralTokenId
        );


        emit Repay(loanId, msg.sender, lender, interest, loan.loanAmount);
        emit Close(loanId);
    }
```

Finally, the new lender receives the lend ticket that has no utility at this point. The borrower now possesses the NFT, original loan amount, and accrued interest.

## Tools Used
Manual review.

## Recommended Mitigation Steps
Move the line to transfer the lend ticket to the new lender above the line to transfer to funds to the original lender. Or, use reentrancyGuard from OpenZeppelin to remove the risk of reentrant calls completely.

If desired, also require that the lender cannot be the same account as the borrower of a loan.

