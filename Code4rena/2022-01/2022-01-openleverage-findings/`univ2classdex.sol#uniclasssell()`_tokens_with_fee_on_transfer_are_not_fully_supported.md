## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`UniV2ClassDex.sol#uniClassSell()` Tokens with fee on transfer are not fully supported](https://github.com/code-423n4/2022-01-openleverage-findings/issues/208) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-openleverage/blob/501e8f5c7ebaf1242572712626a77a3d65bdd3ad/openleverage-contracts/contracts/dex/bsc/UniV2ClassDex.sol#L31-L56

```solidity
function uniClassSell(DexInfo memory dexInfo,
    address buyToken,
    address sellToken,
    uint sellAmount,
    uint minBuyAmount,
    address payer,
    address payee
) internal returns (uint buyAmount){
    address pair = getUniClassPair(buyToken, sellToken, dexInfo.factory);
    IUniswapV2Pair(pair).sync();
    (uint256 token0Reserves, uint256 token1Reserves,) = IUniswapV2Pair(pair).getReserves();
    sellAmount = transferOut(IERC20(sellToken), payer, pair, sellAmount);
    uint balanceBefore = IERC20(buyToken).balanceOf(payee);
    dexInfo.fees = getPairFees(dexInfo, pair);
    if (buyToken < sellToken) {
        buyAmount = getAmountOut(sellAmount, token1Reserves, token0Reserves, dexInfo.fees);
        IUniswapV2Pair(pair).swap(buyAmount, 0, payee, "");
    } else {
        buyAmount = getAmountOut(sellAmount, token0Reserves, token1Reserves, dexInfo.fees);
        IUniswapV2Pair(pair).swap(0, buyAmount, payee, "");
    }

    require(buyAmount >= minBuyAmount, 'buy amount less than min');
    uint bought = IERC20(buyToken).balanceOf(payee).sub(balanceBefore);
    return bought;
}
```

While `uniClassBuy()` correctly checks the actually received amount by comparing the before and after the balance of the receiver, `uniClassSell()` trusted the result given by `getAmountOut()`. This makes `uniClassSell()` can result in an output amount fewer than `minBuyAmount`.

https://github.com/code-423n4/2022-01-openleverage/blob/501e8f5c7ebaf1242572712626a77a3d65bdd3ad/openleverage-contracts/contracts/dex/bsc/UniV2ClassDex.sol#L101-L102

### Recommendation

Change to:

```solidity
function uniClassSell(DexInfo memory dexInfo,
    address buyToken,
    address sellToken,
    uint sellAmount,
    uint minBuyAmount,
    address payer,
    address payee
) internal returns (uint bought){
    address pair = getUniClassPair(buyToken, sellToken, dexInfo.factory);
    IUniswapV2Pair(pair).sync();
    (uint256 token0Reserves, uint256 token1Reserves,) = IUniswapV2Pair(pair).getReserves();
    sellAmount = transferOut(IERC20(sellToken), payer, pair, sellAmount);
    uint balanceBefore = IERC20(buyToken).balanceOf(payee);
    dexInfo.fees = getPairFees(dexInfo, pair);
    if (buyToken < sellToken) {
        buyAmount = getAmountOut(sellAmount, token1Reserves, token0Reserves, dexInfo.fees);
        IUniswapV2Pair(pair).swap(buyAmount, 0, payee, "");
    } else {
        buyAmount = getAmountOut(sellAmount, token0Reserves, token1Reserves, dexInfo.fees);
        IUniswapV2Pair(pair).swap(0, buyAmount, payee, "");
    }
    uint bought = IERC20(buyToken).balanceOf(payee).sub(balanceBefore);
    require(bought >= minBuyAmount, 'buy amount less than min');
}
```

