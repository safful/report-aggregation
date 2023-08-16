## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`Pool.sol#withdrawBorrowedAmount()` Validation of pool status can be done earlier to save gas](https://github.com/code-423n4/2021-12-sublime-findings/issues/127) 

# Handle

WatchPug


# Vulnerability details

Check if `_poolStatus` and `block.timestamp` earlier can avoid unnecessary code execution when this check failed.

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Pool.sol#L310-L348

```solidity
    function withdrawBorrowedAmount() external override onlyBorrower(msg.sender) nonReentrant {
        LoanStatus _poolStatus = poolVariables.loanStatus;
        uint256 _tokensLent = totalSupply();
        require(
            _poolStatus == LoanStatus.COLLECTION &&
                poolConstants.loanStartTime < block.timestamp &&
                block.timestamp < poolConstants.loanWithdrawalDeadline,
            'WBA1'
        );
        IPoolFactory _poolFactory = IPoolFactory(poolFactory);
        require(_tokensLent >= _poolFactory.minBorrowFraction().mul(poolConstants.borrowAmountRequested).div(10**30), 'WBA2');

        poolVariables.loanStatus = LoanStatus.ACTIVE;
        uint256 _currentCollateralRatio = getCurrentCollateralRatio();
        require(_currentCollateralRatio >= poolConstants.idealCollateralRatio, 'WBA3');

        uint256 _noOfRepaymentIntervals = poolConstants.noOfRepaymentIntervals;
        uint256 _repaymentInterval = poolConstants.repaymentInterval;
        IRepayment(_poolFactory.repaymentImpl()).initializeRepayment(
            _noOfRepaymentIntervals,
            _repaymentInterval,
            poolConstants.borrowRate,
            poolConstants.loanStartTime,
            poolConstants.borrowAsset
        );
        IExtension(_poolFactory.extension()).initializePoolExtension(_repaymentInterval);

        address _borrowAsset = poolConstants.borrowAsset;
        (uint256 _protocolFeeFraction, address _collector) = _poolFactory.getProtocolFeeData();
        uint256 _protocolFee = _tokensLent.mul(_protocolFeeFraction).div(10**30);
        delete poolConstants.loanWithdrawalDeadline;

        uint256 _feeAdjustedWithdrawalAmount = _tokensLent.sub(_protocolFee);

        SavingsAccountUtil.transferTokens(_borrowAsset, _protocolFee, address(this), _collector);
        SavingsAccountUtil.transferTokens(_borrowAsset, _feeAdjustedWithdrawalAmount, address(this), msg.sender);

        emit AmountBorrowed(_feeAdjustedWithdrawalAmount, _protocolFee);
    }
```

### Recommendation

Change to:

```solidity
    function withdrawBorrowedAmount() external override onlyBorrower(msg.sender) nonReentrant {
        LoanStatus _poolStatus = poolVariables.loanStatus;
        require(
            _poolStatus == LoanStatus.COLLECTION &&
                poolConstants.loanStartTime < block.timestamp &&
                block.timestamp < poolConstants.loanWithdrawalDeadline,
            'WBA1'
        );
        uint256 _tokensLent = totalSupply();
        IPoolFactory _poolFactory = IPoolFactory(poolFactory);
        require(_tokensLent >= _poolFactory.minBorrowFraction().mul(poolConstants.borrowAmountRequested).div(10**30), 'WBA2');

        poolVariables.loanStatus = LoanStatus.ACTIVE;
        uint256 _currentCollateralRatio = getCurrentCollateralRatio();
        require(_currentCollateralRatio >= poolConstants.idealCollateralRatio, 'WBA3');

        uint256 _noOfRepaymentIntervals = poolConstants.noOfRepaymentIntervals;
        uint256 _repaymentInterval = poolConstants.repaymentInterval;
        IRepayment(_poolFactory.repaymentImpl()).initializeRepayment(
            _noOfRepaymentIntervals,
            _repaymentInterval,
            poolConstants.borrowRate,
            poolConstants.loanStartTime,
            poolConstants.borrowAsset
        );
        IExtension(_poolFactory.extension()).initializePoolExtension(_repaymentInterval);

        address _borrowAsset = poolConstants.borrowAsset;
        (uint256 _protocolFeeFraction, address _collector) = _poolFactory.getProtocolFeeData();
        uint256 _protocolFee = _tokensLent.mul(_protocolFeeFraction).div(10**30);
        delete poolConstants.loanWithdrawalDeadline;

        uint256 _feeAdjustedWithdrawalAmount = _tokensLent.sub(_protocolFee);

        SavingsAccountUtil.transferTokens(_borrowAsset, _protocolFee, address(this), _collector);
        SavingsAccountUtil.transferTokens(_borrowAsset, _feeAdjustedWithdrawalAmount, address(this), msg.sender);

        emit AmountBorrowed(_feeAdjustedWithdrawalAmount, _protocolFee);
    }
```

