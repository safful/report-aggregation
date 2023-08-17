## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- reviewed

# [[WP-M11] `CEthInterface#mint()` reading non-existing returns makes `topUp()` with native token alway revert](https://github.com/code-423n4/2022-04-backd-findings/issues/125) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/interfaces/vendor/CTokenInterfaces.sol#L345


# Vulnerability details

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/interfaces/vendor/CTokenInterfaces.sol#L345

```solidity
 function mint() external payable returns (uint256);
```

`mint()` for native cToken (`CEther`) will return nothing, while the current `CEthInterface` interface defines the returns as `(uint256)`.

In the current implementation, the interface for `CToken` is used for both `CEther` and `CErc20`.

As a result, the transaction will revert with the error: `function returned an unexpected amount of data` when `topUp()` with the native token (ETH).

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/handlers/CompoundHandler.sol#L57-L70

```solidity
    CToken ctoken = cTokenRegistry.fetchCToken(underlying);
    uint256 initialTokens = ctoken.balanceOf(address(this));

    address addr = account.addr();

    if (repayDebt) {
        amount -= _repayAnyDebt(addr, underlying, amount, ctoken);
        if (amount == 0) return true;
    }

    uint256 err;
    if (underlying == address(0)) {
        err = ctoken.mint{value: amount}(amount);
    }
```

Ref:

| method  | CEther | CErc20 |
|----------|------------|-------------|
| mint()   | revert      | error code  |
| redeem() | error code | error code  |
| repayBorrow() | revert | error code  |
| repayBorrowBehalf() | revert | error code  |


- Compound's cToken mint doc: https://compound.finance/docs/ctokens#mint
- Compound CEther.mint() https://github.com/compound-finance/compound-protocol/blob/v2.8.1/contracts/CEther.sol#L46
- Compound CErc20.mint() https://github.com/compound-finance/compound-protocol/blob/v2.8.1/contracts/CErc20.sol#L46

