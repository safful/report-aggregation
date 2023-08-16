## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [IndexPool pow overflows when `weightRatio` > 10.](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/28) 

# Handle

broccoli


# Vulnerability details

## Impact

In the IndexPool contract, pow is used in calculating price. [IndexPool.sol#L255-L266](https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/pool/IndexPool.sol#L255-L266)
However, Pow is easy to cause overflow. If the `weightRatio` is large (e.g. 10), there's always overflow.

Lp providers can still provide liquidity to the pool where no one can swap. All pools need to redeploy. I consider this a high-risk issue.


## Proof of concept

It's easy to trigger this bug by deploying a 1:10 IndexPool.

```python
    deployed_code = encode_abi(["address[]","uint136[]","uint256"], [
        (link.address, dai.address),
        (10**18, 10 * 10**18),
        10**13
    ])
    tx_hash = master_deployer.functions.deployPool(index_pool_factory.address, deployed_code).transact()
```

Transactions would be reverted when buying `link` with `dai`.
## Tools Used

None

## Recommended Mitigation Steps

The `weightRatio` is an 18 decimals number. It should be divided by `(BASE)^exp`. The scale in the contract is not consistent. Recommend the dev to check all the scales/ decimals.


