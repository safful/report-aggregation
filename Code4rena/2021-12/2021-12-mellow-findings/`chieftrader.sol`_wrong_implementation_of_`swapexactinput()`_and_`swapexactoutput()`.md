## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`ChiefTrader.sol` Wrong implementation of `swapExactInput()` and `swapExactOutput()`](https://github.com/code-423n4/2021-12-mellow-findings/issues/108) 

# Handle

WatchPug


# Vulnerability details

When a caller calls `ChiefTrader.sol#swapExactInput()`, it will call `ITrader(traderAddress).swapExactInput()`.

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/trader/ChiefTrader.sol#L59-L59

```solidity=59
return ITrader(traderAddress).swapExactInput(0, amount, recipient, path, options);
```

However, in the current implementation, inputToken is not approved to the `traderAddress`.

For example, in `UniV3Trader.sol#_swapExactInputSingle`, at L89, it tries to transfer inputToken from `msg.sender` (which is `ChiefTrader`), since it's not approved, this will revert.

Plus, the inputToken should also be transferred from the caller before calling the subtrader.

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/trader/UniV3Trader.sol#L89-L89

```solidity=89
    IERC20(input).safeTransferFrom(msg.sender, address(this), amount);
```

The same problem exits in `swapExactOutput()`:

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/trader/ChiefTrader.sol#L63-L75

```solidity=63
function swapExactOutput(
        uint256 traderId,
        uint256 amount,
        address,
        PathItem[] calldata path,
        bytes calldata options
    ) external returns (uint256) {
        require(traderId < _traders.length, TraderExceptionsLibrary.TRADER_NOT_FOUND_EXCEPTION);
        _requireAllowedTokens(path);
        address traderAddress = _traders[traderId];
        address recipient = msg.sender;
        return ITrader(traderAddress).swapExactOutput(0, amount, recipient, path, options);
    }
```

### Recommendation

Approve the inputToken to the subtrader and transfer from the caller before calling `ITrader.swapExactInput()` and `ITrader.swapExactOutput()`.

Or maybe just remove support of `swapExactInput()` and `swapExactOutput()` in `ChiefTrader`.

