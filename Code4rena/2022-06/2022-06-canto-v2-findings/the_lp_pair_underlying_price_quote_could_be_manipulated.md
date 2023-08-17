## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [The LP pair underlying price quote could be manipulated](https://github.com/code-423n4/2022-06-canto-v2-findings/issues/152) 

# Lines of code

https://github.com/Plex-Engineer/lending-market-v2/blob/ea5840de72eab58bec837bb51986ac73712fcfde/contracts/Stableswap/BaseV1-periphery.sol#L522-L526
https://github.com/Plex-Engineer/lending-market-v2/blob/ea5840de72eab58bec837bb51986ac73712fcfde/contracts/Stableswap/BaseV1-periphery.sol#L198-L217


# Vulnerability details

# The LP pair underlying price quote could be manipulated


## Impact

The underlying price for LP pool pair can be manipulated. This kind of price mainpulation happened before, can be found here: [Warp Fincance event](https://rekt.news/warp-finance-rekt/).

Whick may lead to the exploit of the pool by a malicious user.

## Proof of Concept

file: lending-market-v2/contracts/Stableswap/BaseV1-periphery.sol
522-526， 198-217:
```
            uint price0 = (token0 != USDC) ? IBaseV1Pair(pairFor(USDC, token0, stable0)).quote(token0, 1, 8) : 1;
            uint price1 = (token1 != USDC) ? IBaseV1Pair(pairFor(USDC, token1, stable1)).quote(token1, 1, 8) : 1;
            // how much of each asset is 1 LP token redeemable for
            (uint amt0, uint amt1) = quoteRemoveLiquidity(token0, token1, stablePair, 1);
            price = amt0 * price0 + amt1 * price1;


    function quoteRemoveLiquidity(
        address tokenA,
        address tokenB,
        bool stable,
        uint liquidity
    ) public view returns (uint amountA, uint amountB) {
        // create the pair if it doesn"t exist yet
        address _pair = IBaseV1Factory(factory).getPair(tokenA, tokenB, stable);

        if (_pair == address(0)) {
            return (0,0);
        }

        (uint reserveA, uint reserveB) = getReserves(tokenA, tokenB, stable);
        uint _totalSupply = erc20(_pair).totalSupply();

        amountA = liquidity * reserveA / _totalSupply; // using balances ensures pro-rata distribution
        amountB = liquidity * reserveB / _totalSupply; // using balances ensures pro-rata distribution

    }
```

The price of the LP pair is determined by the TVL of the pool, given by:
`amt0 * price0 + amt1 * price1`. However, when a malicious user dumps large amount of any token into the pool, the whole TVL will be significantly increased, which leads to inproper calculation of the price.


## Tools Used
mannual analysis

## Recommended Mitigation Steps

A differenct approach to calculate the LP price can be found [here](https://cmichel.io/pricing-lp-tokens/).


