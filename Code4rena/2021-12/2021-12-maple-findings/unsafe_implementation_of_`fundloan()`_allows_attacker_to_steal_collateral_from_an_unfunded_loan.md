## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Unsafe implementation of `fundLoan()` allows attacker to steal collateral from an unfunded loan](https://github.com/code-423n4/2021-12-maple-findings/issues/47) 

# Handle

WatchPug


# Vulnerability details

https://github.com/maple-labs/loan/blob/9684bcef06481e493d060974b1777a4517c4e792/contracts/MapleLoanInternals.sol#L257-L273

```solidity=257
    uint256 treasuryFee = (fundsLent_ * ILenderLike(lender_).treasuryFee() * _paymentInterval * _paymentsRemaining) / uint256(365 days * 10_000);

    // Transfer delegate fee, if any, to the pool delegate, and decrement drawable funds.
    uint256 delegateFee = (fundsLent_ * ILenderLike(lender_).investorFee() * _paymentInterval * _paymentsRemaining) / uint256(365 days * 10_000);

    // Drawable funds is the amount funded, minus any fees.
    _drawableFunds = fundsLent_ - treasuryFee - delegateFee;

    require(
        treasuryFee == uint256(0) || ERC20Helper.transfer(_fundsAsset, ILenderLike(lender_).mapleTreasury(), treasuryFee),
        "MLI:FL:T_TRANSFER_FAILED"
    );

    require(
        delegateFee == uint256(0) || ERC20Helper.transfer(_fundsAsset, ILenderLike(lender_).poolDelegate(), delegateFee),
        "MLI:FL:PD_TRANSFER_FAILED"
        );
```

In the current implementation, `mapleTreasury`, `poolDelegate` and `treasuryFee` are taken from user input `lender_`, which can be faked by setting up a contract with `ILenderLike` interfaces.

This allows the attacker to set very high fees, making `_drawableFunds` near 0.

Since `mapleTreasury` and `poolDelegate` are also read from `lender_`, `treasuryFee` and `investorFee` can be retrieved back to the attacker.

As a result, the borrower won't get any `_drawableFunds` while also being unable to remove collateral.

### PoC

Given:

- `_collateralRequired` = 10 BTC
- `_principalRequested` = 1,000,000 USDC
- `_paymentInterval` = 1 day
- `_paymentsRemaining` = 10
- `_gracePeriod` = 1 day

1. Alice (borrower) calls `postCollateral()` and added `10 BTC` as `_collateralAsset`;
2. The attacker calls `fundLoan()` by taking `1,000,000 USDC` of flashloan and using a fake `lender`contract;
3. Alice calls `drawdownFunds()` with any amount > 0 will fail;
4. Alice calls `removeCollateral()` with any amount > 0 will get "MLI:DF:INSUFFICIENT_COLLATERAL" error;
5. Unless Alice make payment (which is meaningless), after 2 day, the attacker can call `repossess()` and get `10 BTC`.

### Recommendation

Consider reading `treasuryFee`, `investorFee`, `mapleTreasury`, `poolDelegate` from an authoritative source instead.

