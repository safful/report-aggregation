## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [ERC777 tokens can bypass `depositCap` guard](https://github.com/code-423n4/2022-04-backd-findings/issues/47) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/pool/LiquidityPool.sol#L523


# Vulnerability details

## Impact
When ERC777 token is used as the underlying token for a `LiquidityPool`, a depositor can reenter `depositFor` and bypass the `depositCap` requirement check, resulting in higher total deposit than intended by governance.

## Proof of Concept
- An empty ERC777 liquidity pool is capped at 1.000 token.
- Alice deposits 1.000 token. Before the token is actually sent to the contract, `tokensToSend` ERC777 hook is called and Alice reenters `depositFor`.
- As the previous deposit hasn't been taken into account, the reentrancy passes the `depositCap` check.
- Pool has 2.000 token now, despite the 1.000 deposit cap.

## Recommended Mitigation Steps
Add reentrancy guards to `depositFor`.


