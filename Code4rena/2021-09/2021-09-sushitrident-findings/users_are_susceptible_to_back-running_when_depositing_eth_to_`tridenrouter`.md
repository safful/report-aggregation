## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [Users are susceptible to back-running when depositing ETH to `TridenRouter`](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/179) 

# Handle

broccoli


# Vulnerability details

## Impact

The `_depositToBentoBox` and `_depositFromUserToBentoBox` allow users to provide ETH to the router, which is later deposited to the `bento` contract for swapping other assets or providing liquidity. However, in these two functions, the input parameter does not represent the actual amount of ETH to deposit, and users have to calculate the actual amount and send it to the router, causing a back-run vulnerability if there are ETH left after the operation.

## Proof of Concept

1. A user wants to swap ETH to DAI. He calls `exactInputSingleWithNativeToken` on the router with the corresponding parameters and `params.amountIn` being 10. Before calling the function, he calculates `bento.toAmount(wETH, 10, true) = 15` and thus send 15 ETH to the router.
2. However, at the time when his transaction is executed, `bento.toAmount(wETH, amount, true)` becomes to `14`, which could happen if someone calls `harvest` on `bento` to update the `elastic` value of the `wETH` token.
3. As a result, only 14 ETH is transferred to the pool, and 1 ETH is left in the router. Anyone could back-run the user's transaction to retrieve the remaining 1 ETH from the router by calling the `refundETH` function.

Referenced code:
[TridentRouter.sol#L318-L351](https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/TridentRouter.sol#L318-L351)

## Recommended Mitigation Steps

Directly push the remaining ETH to the sender to prevent any ETH left in the router.

