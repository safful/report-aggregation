## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Pair creation can be denied](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/197) 

# Handle

cmichel


# Vulnerability details

The `LaunchEvent.createPair` requires that no previous pool was created for the `WAVAX <> _token` pair.

```solidity
function createPair() external isStopped(false) atPhase(Phase.PhaseThree) {
    (address wavaxAddress, address tokenAddress) = (
        address(WAVAX),
        address(token)
    );
    // @audit grief: anyone can create pair
    require(
        factory.getPair(wavaxAddress, tokenAddress) == address(0),
        "LaunchEvent: pair already created"
    );

    // ...
}
```

A griefer can create a pool for the `WAVAX <> _token` pair by calling [`JoeFactory.createPair(WAVAX, _token)`](https://snowtrace.io/address/0x9ad6c38be94206ca50bb0d90783181662f0cfa10#contracts) while the launch event phase 1 or 2 is running.
No liquidity can then be provided and an emergency state must be triggered for users and the issuer to be able to withdraw again.

#### Recommendation
It must be assumed that the pool is already created and even initialized as pool creation and liquidity provisioning is permissionless.
Special attention must be paid if the pool is already initialized with liquidity at a different price than the launch event price.

It would be enough to have a standard min. LP return "slippage" check (using parameter values for `amountAMin/amountBMin` instead of the hardcoded ones in `router.addLiquidity`) in `LaunchEvent.createPair()`.
The function must then be callable with special privileges only, for example, by the issuer.
Alternatively, the slippage check can be hardcoded as a percentage of the raised amounts (`amountADesired = 0.95 * wavaxReserve, amountBDesired = 0.95 * tokenAllocated`).

This will prevent attacks that try to provide LP at a bad pool price as the transaction will revert when receiving less than the slippage parameter.
If the pool is already initialized, it should just get arbitraged to the auction token price and liquidity can then be provided at the expected rate again.


