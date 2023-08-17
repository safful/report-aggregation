## Tags

- bug
- duplicate
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-04-backed-findings/issues/103) 

# Gas optimization report

## Keep revert strings below 32 bytes

Strings are stored in slots of 32 bytes, and hence the length of the revert string should be at max 32 bytes to fit inside 1 slot. If the string is just 33 bytes it will occupy 2 slots (64 bytes). Keeping the string size at 32 bytes or below will save gas on both deployment and when the revert condition is met.

Since the used version of Solidity is `>=0.8.4` it would also be worth considering using [Custom Errors](https://blog.soliditylang.org/2021/04/21/custom-errors/) which is both more gas efficient and allows thorough error descriptions using [NatSpec](https://docs.soliditylang.org/en/latest/natspec-format.html).

The relevant code:

```
NFTLoanFacilitator.sol line 84:   "NFTLoanFacilitator: cannot use tickets as collateral"
NFTLoanFacilitator.sol line 86:   "NFTLoanFacilitator: cannot use tickets as collateral"
NFTLoanFacilitator.sol line 118:  "NFTLoanFacilitator: borrow ticket holder only"
NFTLoanFacilitator.sol line 121:  "NFTLoanFacilitator: has lender, use repayAndCloseLoan"
NFTLoanFacilitator.sol line 146:  "NFTLoanFacilitator: rate too high"
NFTLoanFacilitator.sol line 147:  "NFTLoanFacilitator: duration too low"
NFTLoanFacilitator.sol line 148:  "NFTLoanFacilitator: amount too low"
NFTLoanFacilitator.sol line 171:  "NFTLoanFacilitator: rate too high"
NFTLoanFacilitator.sol line 172:  "NFTLoanFacilitator: duration too low"
NFTLoanFacilitator.sol line 178:  "NFTLoanFacilitator: proposed terms must be better than existing terms"
NFTLoanFacilitator.sol line 189:  "NFTLoanFacilitator: accumulated interest exceeds uint128"
NFTLoanFacilitator.sol line 255:  "NFTLoanFacilitator: lend ticket holder only"
NFTLoanFacilitator.sol line 259:  "NFTLoanFacilitator: payment is not late"
NFTLoanFacilitator.sol line 321:  "NFTLoanFacilitator: 0 improvement rate"
NFTLoanFacilitator.sol line 86:

NFTLoanTicket.sol line 15:        "NFTLoanTicket: only loan facilitator"
```

## Cache loanAmount

In the following code the variable `loanInfo[loanId].loanAmount` is read from storage 3 times (see audit-info comments), and hence should be cached so it is only read from storage once.

```solidity
function repayAndCloseLoan(uint256 loanId) external override notClosed(loanId) {
        Loan storage loan = loanInfo[loanId];

        uint256 interest = _interestOwed(
            loan.loanAmount, @audit-info SLOAD1
            loan.lastAccumulatedTimestamp,
            loan.perAnumInterestRate,
            loan.accumulatedInterest
        );
        address lender = IERC721(lendTicketContract).ownerOf(loanId);
        loan.closed = true;
        ERC20(loan.loanAssetContractAddress).safeTransferFrom(msg.sender, lender, interest + loan.loanAmount); @audit-info SLOAD2
        IERC721(loan.collateralContractAddress).safeTransferFrom(
            address(this),
            IERC721(borrowTicketContract).ownerOf(loanId),
            loan.collateralTokenId
        );

        emit Repay(loanId, msg.sender, lender, interest, loan.loanAmount); @audit-info SLOAD3
        emit Close(loanId);
    }
```

Change this into 

```solidity
function repayAndCloseLoan(uint256 loanId) external override notClosed(loanId) {
        Loan storage loan = loanInfo[loanId];
        uint128 loanAmount = loan.loanAmount;

        uint256 interest = _interestOwed(
            loanAmount,
            loan.lastAccumulatedTimestamp,
            loan.perAnumInterestRate,
            loan.accumulatedInterest
        );
        address lender = IERC721(lendTicketContract).ownerOf(loanId);
        loan.closed = true;
        ERC20(loan.loanAssetContractAddress).safeTransferFrom(msg.sender, lender, interest + loanAmount);
        IERC721(loan.collateralContractAddress).safeTransferFrom(
            address(this),
            IERC721(borrowTicketContract).ownerOf(loanId),
            loan.collateralTokenId
        );

        emit Repay(loanId, msg.sender, lender, interest, loanAmount);
        emit Close(loanId);
    }
```

The change is passing the provided test suite, and the `.gas-snapshot` reflected the change by reducing the gas costs from:

```solidity
NFTLoanFacilitatorGasBenchMarkTest:testRepayAndClose() (gas: 81320 -> 81064)
NFTLoanFacilitatorTest:testRepayAndCloseSuccessful() (gas: 447725 -> 447469)
NFTLoanFacilitatorTest:testRepayInterestOwedExceedingUint128() (gas: 465901 -> 465645)
```