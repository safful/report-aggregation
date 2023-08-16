## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Remove unnecessary variables can make the code simpler and save some gas](https://github.com/code-423n4/2021-12-sublime-findings/issues/112) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L691-L727

```solidity=691{693-694}
function borrow(uint256 _id, uint256 _amount) external payable nonReentrant onlyCreditLineBorrower(_id) {
    require(creditLineVariables[_id].status == CreditLineStatus.ACTIVE, 'CreditLine: The credit line is not yet active.');
    uint256 _borrowableAmount = calculateBorrowableAmount(_id);
    require(_amount <= _borrowableAmount, "CreditLine::borrow - The current collateral ratio doesn't allow to withdraw the amount");
    address _borrowAsset = creditLineConstants[_id].borrowAsset;
    address _lender = creditLineConstants[_id].lender;
```

`_borrowableAmount` is unnecessary. The code above can be changed to:

```solidity=691{693}
function borrow(uint256 _id, uint256 _amount) external payable nonReentrant onlyCreditLineBorrower(_id) {
    require(creditLineVariables[_id].status == CreditLineStatus.ACTIVE, 'CreditLine: The credit line is not yet active.');
    require(_amount <= calculateBorrowableAmount(_id), "CreditLine::borrow - The current collateral ratio doesn't allow to withdraw the amount");
    address _borrowAsset = creditLineConstants[_id].borrowAsset;
    address _lender = creditLineConstants[_id].lender;
```

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L391-L399

```solidity=391
function calculateInterest(
    uint256 _principal,
    uint256 _borrowRate,
    uint256 _timeElapsed
) public pure returns (uint256) {
    uint256 _interest = _principal.mul(_borrowRate).mul(_timeElapsed).div(10**30).div(YEAR_IN_SECONDS);

    return _interest;
}
```

`_interest` is unnecessary.

