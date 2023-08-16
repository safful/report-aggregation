## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Functions calculating the value of `BPT` is vulnerable to flash-loan attacks.](https://github.com/code-423n4/2021-04-maple-findings/issues/113) 

# Handle

shw


# Vulnerability details

## Impact

In `library/PoolLib.sol`, the return value of functions `BPTVal` and `getPoolSharesRequired` are vulnerable by flash-loan attacks. The attacker can inflate the results of these two functions by swapping a large amount of `liquidityAsset` into the pool and swaps back after the functions are called to deceive the pool contract that BPT has a relatively high price.

Although currently `BPTVal` is not used and `getPoolSharesRequired` only affects the required staking amounts of token for a pool delegate, the code is vulnerable and could be misused by anyone in the future.

## Proof of Concept

In the function `BPTVal`, the value of BPT in units of liquidityAsset is calculated directly from the balance of `liquidityAsset` in the Balancer pool (`PoolLib.sol#331`). For function `getPoolSharesRequired`, the required BPT to be burned also depends on the current balance of `liquidityAsset` in the pool.

## Tools Used

None

## Recommended Mitigation Steps

Use the balance of `liquidityAsset` in the previous block to eliminate the possibility of suffering from a flash-loan attack. A time-weight average price can also mitigate the problem.

