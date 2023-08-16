## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [Wrong calculation of `erc20Delta` and `ethDelta`](https://github.com/code-423n4/2021-10-tally-findings/issues/34) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-tally/blob/c585c214edb58486e0564cb53d87e4831959c08b/contracts/swap/Swap.sol#L200-L225

```solidity
function fillZrxQuote(
    IERC20 zrxBuyTokenAddress,
    address payable zrxTo,
    bytes calldata zrxData,
    uint256 ethAmount
) internal returns (uint256, uint256) {
    uint256 originalERC20Balance = 0;
    if(!signifiesETHOrZero(address(zrxBuyTokenAddress))) {
        originalERC20Balance = zrxBuyTokenAddress.balanceOf(address(this));
    }
    uint256 originalETHBalance = address(this).balance;

    (bool success,) = zrxTo.call{value: ethAmount}(zrxData);
    require(success, "Swap::fillZrxQuote: Failed to fill quote");

    uint256 ethDelta = address(this).balance.subOrZero(originalETHBalance);
    uint256 erc20Delta;
    if(!signifiesETHOrZero(address(zrxBuyTokenAddress))) {
        erc20Delta = zrxBuyTokenAddress.balanceOf(address(this)).subOrZero(originalERC20Balance);
        require(erc20Delta > 0, "Swap::fillZrxQuote: Didn't receive bought token");
    } else {
        require(ethDelta > 0, "Swap::fillZrxQuote: Didn't receive bought ETH");
    }

    return (erc20Delta, ethDelta);
}
```

When a user tries to swap unwrapped ETH to ERC20, even if there is a certain amount of ETH refunded, at L215, `ethDelta` will always be `0`. 

That's because `originalETHBalance` already includes the `msg.value` sent by the caller.

Let's say the ETH balance of the contract is `1 ETH` before the swap.

- A user swaps `10 ETH` to USDC;
- `originalETHBalance` will be `11 ETH`;
- If there is `1 ETH` of refund;
- `ethDelta` will be `0` as the new balance is `2 ETH` and `subOrZero(2, 11)` is `0`.

Similarly, `erc20Delta` is also computed wrong.

Consider a special case of a user trying to arbitrage from `WBTC` to `WBTC`, the `originalERC20Balance` already includes the input amount, `erc20Delta` will always be much lower than the actual delta amount.

For example, for an arb swap from `1 WBTC` to `1.1 WBTC`, the `ethDelta` will be `0.1 WBTC` while it should be `1.1 WBTC`.

### Impact

- User can not get ETH refund for swaps from ETH to ERC20 tokens;
- Arb swap with the same input and output token will suffer the loss of almost all of their input amount unexpectedly.

### Recommendation

Consider subtracting the input amount from the originalBalance.

