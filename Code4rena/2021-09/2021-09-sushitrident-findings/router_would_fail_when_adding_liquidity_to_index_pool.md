## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Router would fail when adding liquidity to index Pool](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/68) 

# Handle

broccoli


# Vulnerability details

## Impact

TridentRouter is easy to fail when trying to provide liquidity to an index pool.

Users would not get extra lp if they are not providing lp at the pool's spot price. It's the same design as uniswap v2. However, uniswap's v2 handle's the dirty part. 

[UniswapV2Router02.sol#L61-L76](https://github.com/Uniswap/v2-periphery/blob/master/contracts/UniswapV2Router02.sol#L61-L76)
Users would not lose tokens if they use the router.

However, the router wouldn't stop users from transferring extra tokens.
[TridentRouter.sol#L168-L190](https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/TridentRouter.sol#L168-L190)

Second, the price would possibly change when the transaction is confirmed. This would be reverted in the index pool. 

Users would either transfer extra tokens or fail. I consider this is a medium-risk issue.


## Proof of Concept

[TridentRouter.sol#L168-L190](https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/TridentRouter.sol#L168-L190)

A possible scenario:

There's a BTC/USD pool. BTC = 50000 USD.
1. A user sends a transaction to transfer 1 BTC and 50000 USD.
2. After the user send a transaction, a random bot buying BTC with USD.
3. The transaction at step 1 is mined. Since the BTC price is not 50000 USD, the transaction fails.


## Tools Used

None

## Recommended Mitigation Steps

Please refer to the uniswap v2 router.
[UniswapV2Router02.sol#L61-L76](https://github.com/Uniswap/v2-periphery/blob/master/contracts/UniswapV2Router02.sol#L61-L76)

The router should calculate the optimal parameters for users.



