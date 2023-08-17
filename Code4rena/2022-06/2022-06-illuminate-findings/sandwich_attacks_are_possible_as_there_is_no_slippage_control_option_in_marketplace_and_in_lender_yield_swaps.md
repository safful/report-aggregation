## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Sandwich attacks are possible as there is no slippage control option in Marketplace and in Lender yield swaps](https://github.com/code-423n4/2022-06-illuminate-findings/issues/389) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/marketplace/MarketPlace.sol#L131-L189
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L634-L657


# Vulnerability details

Swapping function in Marketplace and Lender's yield() can be sandwiched as there is no slippage control option. Trades can happen at a manipulated price and end up receiving fewer tokens than current market price dictates.

Placing severity to be medium as those are system core operations, while funds there can be substantial, so sandwich attacks are often enough economically viable and thus probable, while they result in a partial fund loss.

## Proof of Concept

All four swapping functions of Marketplace do not allow for slippage control:

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/marketplace/MarketPlace.sol#L131-L189

```solidity
    /// @notice sells the PT for the PT via the pool
    /// @param u address of the underlying asset
    /// @param m maturity (timestamp) of the market
    /// @param a amount of PT to swap
    /// @return uint128 amount of PT bought
    function sellPrincipalToken(
        address u,
        uint256 m,
        uint128 a
    ) external returns (uint128) {
        IPool pool = IPool(pools[u][m]);
        Safe.transfer(IERC20(address(pool.fyToken())), address(pool), a);
        return pool.sellFYToken(msg.sender, pool.sellFYTokenPreview(a));
    }

    /// @notice buys the underlying for the PT via the pool
    /// @param u address of the underlying asset
    /// @param m maturity (timestamp) of the market
    /// @param a amount of underlying tokens to sell
    /// @return uint128 amount of PT received
    function buyPrincipalToken(
        address u,
        uint256 m,
        uint128 a
    ) external returns (uint128) {
        IPool pool = IPool(pools[u][m]);
        Safe.transfer(IERC20(address(pool.base())), address(pool), a);
        return pool.buyFYToken(msg.sender, pool.buyFYTokenPreview(a), a);
    }

    /// @notice sells the underlying for the PT via the pool
    /// @param u address of the underlying asset
    /// @param m maturity (timestamp) of the market
    /// @param a amount of underlying to swap
    /// @return uint128 amount of underlying sold
    function sellUnderlying(
        address u,
        uint256 m,
        uint128 a
    ) external returns (uint128) {
        IPool pool = IPool(pools[u][m]);
        Safe.transfer(IERC20(address(pool.base())), address(pool), a);
        return pool.sellBase(msg.sender, pool.sellBasePreview(a));
    }

    /// @notice buys the underlying for the PT via the pool
    /// @param u address of the underlying asset
    /// @param m maturity (timestamp) of the market
    /// @param a amount of PT to swap
    /// @return uint128 amount of underlying bought
    function buyUnderlying(
        address u,
        uint256 m,
        uint128 a
    ) external returns (uint128) {
        IPool pool = IPool(pools[u][m]);
        Safe.transfer(IERC20(address(pool.fyToken())), address(pool), a);
        return pool.buyBase(msg.sender, pool.buyBasePreview(a), a);
    }
```

Similarly, Lender's yield does the swapping without the ability to control the slippage:

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L634-L657

```solidity
    /// @notice transfers excess funds to yield pool after principal tokens have been lent out
    /// @dev this method is only used by the yield, illuminate and swivel protocols
    /// @param u address of an underlying asset
    /// @param y the yield pool to lend to
    /// @param a the amount of underlying tokens to lend
    /// @param r the receiving address for PTs
    /// @return uint256 the amount of tokens sent to the yield pool
    function yield(
        address u,
        address y,
        uint256 a,
        address r
    ) internal returns (uint256) {
        // preview exact swap slippage on yield
        uint128 returned = IYield(y).sellBasePreview(Cast.u128(a));

        // send the remaing amount to the given yield pool
        Safe.transfer(IERC20(u), y, a);

        // lend out the remaining tokens in the yield pool
        IYield(y).sellBase(r, returned);

        return returned;
    }
```

## Recommended Mitigation Steps

Consider adding minimum accepted return argument to the five mentioned functions and condition execution success on it so the caller can control for the realized slippage and sustain the sandwich attacks to an extent.

