## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved

# [`PostAuctionLauncher.sol#finalize()` Adding liquidity to an existing pool may allows the attacker to steal most of the tokens](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/14) 

# Handle

WatchPug


# Vulnerability details

`PostAuctionLauncher.finalize()` can be called by anyone, and it sends tokens directly to the pair pool to mint liquidity, even when the pair pool exists.

An attacker may control the LP price by creating the pool and then call `finalize()` to mint LP token with unfair price (pay huge amounts of tokens and get few amounts of LP token), and then remove the initial liquidity they acquired when creating the pool and takeout huge amounts of tokens.

https://github.com/sushiswap/miso/blob/2cdb1486a55ded55c81898b7be8811cb68cfda9e/contracts/Liquidity/PostAuctionLauncher.sol#L257

```solidity=216
/**
 * @notice Finalizes Token sale and launches LP.
 * @return liquidity Number of LPs.
 */
function finalize() external nonReentrant returns (uint256 liquidity) {
    // GP: Can we remove admin, let anyone can finalise and launch?
    // require(hasAdminRole(msg.sender) || hasOperatorRole(msg.sender), "PostAuction: Sender must be operator");
    require(marketConnected(), "PostAuction: Auction must have this launcher address set as the destination wallet");
    require(!launcherInfo.launched);

    if (!market.finalized()) {
        market.finalize();
    }
    require(market.finalized());

    launcherInfo.launched = true;
    if (!market.auctionSuccessful() ) {
        return 0;
    }

    /// @dev if the auction is settled in weth, wrap any contract balance 
    uint256 launcherBalance = address(this).balance;
    if (launcherBalance > 0 ) {
        IWETH(weth).deposit{value : launcherBalance}();
    }
    
    (uint256 token1Amount, uint256 token2Amount) =  getTokenAmounts();

    /// @dev cannot start a liquidity pool with no tokens on either side
    if (token1Amount == 0 || token2Amount == 0 ) {
        return 0;
    }

    address pair = factory.getPair(address(token1), address(token2));
    if(pair == address(0)) {
        createPool();
    }

    /// @dev add liquidity to pool via the pair directly
    _safeTransfer(address(token1), tokenPair, token1Amount);
    _safeTransfer(address(token2), tokenPair, token2Amount);
    liquidity = IUniswapV2Pair(tokenPair).mint(address(this));
    launcherInfo.liquidityAdded = BoringMath.to128(uint256(launcherInfo.liquidityAdded).add(liquidity));

    /// @dev if unlock time not yet set, add it.
    if (launcherInfo.unlock == 0 ) {
        launcherInfo.unlock = BoringMath.to64(block.timestamp + uint256(launcherInfo.locktime));
    }
    emit LiquidityAdded(liquidity);
}
```


In line 257, `PostAuctionLauncher` will mint LP with token1Amount and token2Amount. The amounts (token1Amount and token2Amount) are computed according to the auction result, without considering the current price (reserves) of the existing `tokenPair`.

See [PostAuctionLauncher.getTokenAmounts()](https://github.com/sushiswap/miso/blob/2cdb1486a55ded55c81898b7be8811cb68cfda9e/contracts/Liquidity/PostAuctionLauncher.sol#L268)

`PostAuctionLauncher` will receive an unfairly low amount of lp token because the amounts sent to `tokenPair` didn't match the current price of the pair.

See [UniswapV2Pair.mint(...)](https://github.com/sushiswap/miso/blob/2cdb1486a55ded55c81898b7be8811cb68cfda9e/contracts/UniswapV2/UniswapV2Pair.sol#L135)
```solidity=135
liquidity = MathUniswap.min(amount0.mul(_totalSupply) / _reserve0, amount1.mul(_totalSupply) / _reserve1);
```

## Impact

Lose a majority share of the tokens.

## Proof of Concept

1. The attacker creates LP with 0.0000001 token1 and 1000 token2, receives 0.01 LP token;
2. Call `PostAuctionLauncher.finalize()`. PostAuctionLauncher will mint liquidity with 2000 token1 and 1000 token2 for example, receives only  0.01 LP token;
3. The attacker removes all his LP, receives 1000 token1 (most of which come from PostAuctionLauncher).

## Recommended Mitigation Steps

To only support tokenPair created by PostAuctionLauncher or check for the token price before mint liquidity.

