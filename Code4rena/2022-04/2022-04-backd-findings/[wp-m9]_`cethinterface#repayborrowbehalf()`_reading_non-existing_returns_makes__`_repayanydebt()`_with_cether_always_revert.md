## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- reviewed

# [[WP-M9] `CEthInterface#repayBorrowBehalf()` reading non-existing returns makes  `_repayAnyDebt()` with CEther always revert](https://github.com/code-423n4/2022-04-backd-findings/issues/121) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/interfaces/vendor/CTokenInterfaces.sol#L355-L358


# Vulnerability details

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/interfaces/vendor/CTokenInterfaces.sol#L355-L358

```solidity
function repayBorrowBehalf(address borrower, uint256 repayAmount)
        external
        payable
        returns (uint256);
```

`repayBorrowBehalf()` for native cToken (`CEther`) will return nothing, while the current `CEthInterface` interface defines the returns as `(uint256)`.

As a result, `ether.repayBorrowBehalf()` will always revert

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/handlers/CompoundHandler.sol#L117-L118

```solidity
    CEther cether = CEther(address(ctoken));
    err = cether.repayBorrowBehalf{value: debt}(account);
```

Ref:

| method  | CEther | CErc20 |
|----------|------------|-------------|
| mint()   | revert      | error code  |
| redeem() | error code | error code  |
| repayBorrow() | revert | error code  |
| repayBorrowBehalf() | revert | error code  |


- Compound cToken Repay Borrow Behalf doc: https://compound.finance/docs/ctokens#repay-borrow-behalf
- Compound CEther.repayBorrowBehalf() https://github.com/compound-finance/compound-protocol/blob/v2.8.1/contracts/CEther.sol#L92-L95
- Compound CErc20.repayBorrowBehalf() https://github.com/compound-finance/compound-protocol/blob/v2.8.1/contracts/CErc20.sol#L94-L97

