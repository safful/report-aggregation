## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`_investWETHToBuyNOTE` is unnecessarily roundabout.](https://github.com/code-423n4/2022-01-notional-findings/issues/65) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
Gas costs

## Proof of Concept

TreasuryManager has a `_investWETHToBuyNOTE` function which deposits WETH stored on the contract into the NOTE-WETH Balancer pool.

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/TreasuryManager.sol#L168-L211

Note there's a bit of a disconnect between the function name and what it's actually doing. You could argue that you're buying NOTE as you'll end up with an 80% note position but I think it's  more helpful for the purposes of this function to think of it as a "trade" of WETH for BPT.

The current function is as so:
```
IPriceOracle.OracleAverageQuery[] memory queries = new IPriceOracle.OracleAverageQuery[](1);

queries[0].variable = IPriceOracle.Variable.PAIR_PRICE;
queries[0].secs = 3600; // last hour
queries[0].ago = 0; // now

// Gets the balancer time weighted average price denominated in ETH
uint256 noteOraclePrice = IPriceOracle(address(BALANCER_POOL_TOKEN))
    .getTimeWeightedAverage(queries)[0];

BALANCER_VAULT.joinPool(
    NOTE_ETH_POOL_ID,
    address(this),
    sNOTE, // sNOTE will receive the BPT
    IVault.JoinPoolRequest(
        assets,
        maxAmountsIn,
        abi.encode(
            IVault.JoinKind.EXACT_TOKENS_IN_FOR_BPT_OUT,
            maxAmountsIn,
            0 // Accept however much BPT the pool will give us
        ),
        false // Don't use internal balances
    )
);

uint256 noteSpotPrice = _getNOTESpotPrice();

// Calculate the max spot price based on the purchase limit
uint256 maxPrice = noteOraclePrice +
    (noteOraclePrice * notePurchaseLimit) /
    NOTE_PURCHASE_LIMIT_PRECISION;
```

In this function we query the recent price average between NOTE and ETH, perform an unconditional join and then check that the spot price of NOTE in terms of ETH is below some maximum value to ensure that the pool's balances haven't been manipulated such that ETH is being undervalued.

This seems like a fairly roundabout method to set a slippage limit on a join which would be simpler if we queried the exchange rate between BPT and WETH. That allows us to specify a minimum amount of BPT we'd accept.

We'd then avoid using the function `_getNOTESpotPrice` entirely and could save the costs of querying the pool's balances again.

## Recommended Mitigation Steps

Consider changing to something along the lines of

```
IPriceOracle.OracleAverageQuery[] memory queries = new IPriceOracle.OracleAverageQuery[](1);

// Note we're querying the BPT price rather than the pair price now
queries[0].variable = IPriceOracle.Variable.BPT_PRICE;
queries[0].secs = 3600; // last hour
queries[0].ago = 0; // now

// Gets the balancer time weighted average price denominated in ETH
uint256 bptOraclePrice = IPriceOracle(address(BALANCER_POOL_TOKEN))
    .getTimeWeightedAverage(queries)[0];

uint256 minBptOut = (bptOraclePrice * notePurchaseLimit) /  NOTE_PURCHASE_LIMIT_PRECISION;

BALANCER_VAULT.joinPool(
    NOTE_ETH_POOL_ID,
    address(this),
    sNOTE, // sNOTE will receive the BPT
    IVault.JoinPoolRequest(
        assets,
        maxAmountsIn,
        abi.encode(
            IVault.JoinKind.EXACT_TOKENS_IN_FOR_BPT_OUT,
            maxAmountsIn,
            minBptOut
        ),
        false // Don't use internal balances
    )
);
```

We're directly enforcing a slippage limit on the WETH -> BPT conversion rather than doing it in a roundabout way so it's easier to reason about. We save having to query the pool's balances again so save gas and also in the case where the slippage limit is triggered it'll be hit earlier, again saving gas.

