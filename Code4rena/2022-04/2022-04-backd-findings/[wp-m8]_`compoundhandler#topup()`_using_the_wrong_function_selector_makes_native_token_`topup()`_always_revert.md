## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [[WP-M8] `CompoundHandler#topUp()` Using the wrong function selector makes native token `topUp()` always revert](https://github.com/code-423n4/2022-04-backd-findings/issues/120) 

# Lines of code

https://github.com/compound-finance/compound-protocol/blob/v2.8.1/contracts/CEther.sol#L44-L47


# Vulnerability details

https://github.com/compound-finance/compound-protocol/blob/v2.8.1/contracts/CEther.sol#L44-L47

```solidity
function mint() external payable {
    (uint err,) = mintInternal(msg.value);
    requireNoError(err, "mint failed");
}
```

`mint()` for native cToken (`CEther`) does not have any parameters, as the `Function Selector` is based on `the function name with the parenthesised list of parameter types`, when you add a nonexisting `parameter`, the `Function Selector` will be incorrect.

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/interfaces/vendor/CTokenInterfaces.sol#L316

```solidity
    function mint(uint256 mintAmount) external payable virtual returns (uint256);
```

The current implementation uses the same `CToken` interface for both `CEther` and `CErc20` in `topUp()`, and `function mint(uint256 mintAmount)` is a nonexisting function for `CEther`.

As a result, the native token `topUp()` always revert.

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

See also:

- Compound's cToken mint doc: https://compound.finance/docs/ctokens#mint

