## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`customPrecisionMultipliers` would be rounded to zero and break the pool](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/183) 

# Handle

jonah1005


# Vulnerability details

## Impact
CustomPrecisionMultipliers are set in the constructor:
```solidity
        customPrecisionMultipliers[0] = targetPriceStorage.originalPrecisionMultipliers[0].mul(_targetPrice).div(10 ** 18);
```
`originalPrecisionMultipliers` equal to 1 if the token's decimal = 18. The targe price could only be an integer.

If the target price is bigger than 10**18, the user can deposit and trade in the pool. Though, the functionality would be far from the spec.

If the target price is set to be smaller than 10**18, the pool would be broken and all funds would be stuck.

I consider this is a high-risk issue.


## Proof of Concept
Please refer to the implementation.
[Swap.sol#L184-L187](https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/Swap.sol#L184-L187)

We can also trigger the bug by setting a pool with target price = 0.5. (0.5 * 10**18)
## Tools Used
None
## Recommended Mitigation Steps
I recommend providing extra 10**18 in both multipliers.
```solidity
        customPrecisionMultipliers[0] = targetPriceStorage.originalPrecisionMultipliers[0].mul(_targetPrice).mul(10**18).div(10 ** 18);
        customPrecisionMultipliers[1] = targetPriceStorage.originalPrecisionMultipliers[1].mul(10**18);
```
The customswap only supports two tokens in a pool, there's should be enough space. Recommend the devs to go through the trade-off saddle finance has paid to support multiple tokens. The code could be more clean and efficient if the pools' not support multiple tokens.


