## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Wrong formula when add fee `incentivePool` can lead to loss of funds.](https://github.com/code-423n4/2022-03-biconomy-findings/issues/38) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/LiquidityPool.sol#L319-L322


# Vulnerability details

## Impact
The `getAmountToTransfer` function of `LiquidityPool` updates `incentivePool[tokenAddress]` by adding some fee to it but the formula is wrong and the value of `incentivePool[tokenAddress]` will be divided by `BASE_DIVISOR` (10000000000) each time.
After just a few time, the value of `incentivePool[tokenAddress]` will become zero and that amount of `tokenAddress` token will be locked in contract.
## Proof of concept
Line 319-322
```
incentivePool[tokenAddress] = (incentivePool[tokenAddress] + (amount * (transferFeePerc - tokenManager.getTokensInfo(tokenAddress).equilibriumFee))) / BASE_DIVISOR;
```
Let `x = incentivePool[tokenAddress]`, `y = amount`, `z = transferFeePerc` and `t = tokenManager.getTokensInfo(tokenAddress).equilibriumFee`. Then that be written as
```
x = (x + (y * (z - t))) / BASE_DIVISOR;
x = x / BASE_DIVISOR + (y * (z - t)) / BASE_DIVISOR;
```
## Recommended Mitigation Steps
Fix the bug by change line 319-322 to:
```
incentivePool[tokenAddress] += (amount * (transferFeePerc - tokenManager.getTokensInfo(tokenAddress).equilibriumFee)) / BASE_DIVISOR;
```


