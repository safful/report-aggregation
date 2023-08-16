## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[WP-H14] `LiquidityProviders.sol` The share price of the LP can be manipulated and making future liquidityProviders unable to `removeLiquidity()`](https://github.com/code-423n4/2022-03-biconomy-findings/issues/139) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityProviders.sol#L345-L362


# Vulnerability details

https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityProviders.sol#L345-L362

```solidity
function removeLiquidity(uint256 _nftId, uint256 _amount)
    external
    nonReentrant
    onlyValidLpToken(_nftId, _msgSender())
    whenNotPaused
{
    (address _tokenAddress, uint256 nftSuppliedLiquidity, uint256 totalNFTShares) = lpToken.tokenMetadata(_nftId);
    require(_isSupportedToken(_tokenAddress), "ERR__TOKEN_NOT_SUPPORTED");

    require(_amount != 0, "ERR__INVALID_AMOUNT");
    require(nftSuppliedLiquidity >= _amount, "ERR__INSUFFICIENT_LIQUIDITY");
    whiteListPeriodManager.beforeLiquidityRemoval(_msgSender(), _tokenAddress, _amount);
    // Claculate how much shares represent input amount
    uint256 lpSharesForInputAmount = _amount * getTokenPriceInLPShares(_tokenAddress);

    // Calculate rewards accumulated
    uint256 eligibleLiquidity = sharesToTokenAmount(totalNFTShares, _tokenAddress);
```

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/LiquidityProviders.sol#L192-L194

```solidity=192
function sharesToTokenAmount(uint256 _shares, address _tokenAddress) public view returns (uint256) {
    return (_shares * totalReserve[_tokenAddress]) / totalSharesMinted[_tokenAddress];
}
```

The share price of the liquidity can be manipulated to an extremely low value (1 underlying token worth a huge amount of shares), making it possible for `sharesToTokenAmount(totalNFTShares, _tokenAddress)` to overflow in `removeLiquidity()` and therefore freeze users' funds.


### PoC

1. Alice `addTokenLiquidity()` with `1e8 * 1e18` XYZ on B-Chain, totalSharesMinted == `1e44`;
2. Alice `sendFundsToUser()` and bridge `1e8 * 1e18` XYZ from B-Chain to A-Chain;
3. Alice `depositErc20()` and bridge `1e8 * 1e18` XYZ from A-Chain to B-Chain;
4. Alice `removeLiquidity()` and withdraw `1e8 * 1e18 - 1` XYZ, then: `totalReserve` == `1 wei` XYZ, and `totalSharesMinted` == `1e26`;
5. Bob `addTokenLiquidity()` with `3.4e7 * 1e18` XYZ;
6. Bob tries to `removeLiquidity()`.

Expected Results: Bob to get back the deposits;

Actual Results: The tx reverted due to overflow at `sharesToTokenAmount()`.

### Recommendation

https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityProviders.sol#L280-L292

```solidity=280
function _increaseLiquidity(uint256 _nftId, uint256 _amount) internal onlyValidLpToken(_nftId, _msgSender()) {
    (address token, uint256 totalSuppliedLiquidity, uint256 totalShares) = lpToken.tokenMetadata(_nftId);

    require(_amount > 0, "ERR__AMOUNT_IS_0");
    whiteListPeriodManager.beforeLiquidityAddition(_msgSender(), token, _amount);

    uint256 mintedSharesAmount;
    // Adding liquidity in the pool for the first time
    if (totalReserve[token] == 0) {
        mintedSharesAmount = BASE_DIVISOR * _amount;
    } else {
        mintedSharesAmount = (_amount * totalSharesMinted[token]) / totalReserve[token];
    }
    ...
```

Consider locking part of the first mint's liquidity to maintain a minimum amount of `totalReserve[token]`, so that the share price can not be easily manipulated.

