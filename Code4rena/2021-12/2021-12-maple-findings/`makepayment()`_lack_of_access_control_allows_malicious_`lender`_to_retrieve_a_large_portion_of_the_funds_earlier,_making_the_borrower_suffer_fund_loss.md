## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`makePayment()` Lack of access control allows malicious `lender` to retrieve a large portion of the funds earlier, making the borrower suffer fund loss](https://github.com/code-423n4/2021-12-maple-findings/issues/56) 

# Handle

WatchPug


# Vulnerability details

https://github.com/maple-labs/loan/blob/9684bcef06481e493d060974b1777a4517c4e792/contracts/MapleLoan.sol#L86-L93

```solidity=86
function makePayment(uint256 amount_) external override returns (uint256 principal_, uint256 interest_) {
        // The amount specified is an optional amount to be transfer from the caller, as a convenience for EOAs.
        require(amount_ == uint256(0) || ERC20Helper.transferFrom(_fundsAsset, msg.sender, address(this), amount_), "ML:MP:TRANSFER_FROM_FAILED");

        ( principal_, interest_ ) = _makePayment();

        emit PaymentMade(principal_, interest_);
    }
```

The current implementation allows anyone to call `makePayment()` and repay the loan with `_drawableFunds`.

This makes it possible for a malicious `lender` to call `makePayment()` multiple times right after `fundLoan()` and retrieve most of the funds back immediately, while then `borrower` must continue to make payments or lose the `collateral`.

### PoC 

Given:

- `_collateralRequired` = 1 BTC
- `_principalRequested` = 12,000 USDC
- `_paymentInterval` = 30 day
- `_paymentsRemaining` = 12
- `_gracePeriod` = 1 day
- `interestRate_` = 2e17

1. The borrower calls `postCollateral()` and added `1 BTC` as `_collateralAsset`;
2. The lender calls `fundLoan()` and added `12,000 USDC` as  `_fundsAsset`;
3. The lender calls `makePayment()` 11 times, then:
- `_drawableFunds` = 96
- `_claimableFunds` = 11903
- `_principal` = 1553

4. The lender calls `_claimFunds()` get 11,903 USDC of `_fundsAsset` back;

Now, for the borrower `1,579 USDC` is due, but only `96 USDC` can be used. The borrower is now forced to pay the interests for the funds that never be used or lose the collateral.

### Recommendation

Change to:

```solidity=86
function makePayment(uint256 amount_) external override returns (uint256 principal_, uint256 interest_) {
        // The amount specified is an optional amount to be transfer from the caller, as a convenience for EOAs.
        require(amount_ == uint256(0) || ERC20Helper.transferFrom(_fundsAsset, msg.sender, address(this), amount_), "ML:MP:TRANSFER_FROM_FAILED");

        require(msg.sender == _borrower, "ML:DF:NOT_BORROWER");
    
        ( principal_, interest_ ) = _makePayment();

        emit PaymentMade(principal_, interest_);
    }
```

