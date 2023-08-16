## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Numerous gas optimizations in SwapUtils.sol](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/6) 

# Handle

tqts


# Vulnerability details

## Impact
Several references to storage can be cached to save significant amounts of gas.

## Proof of Concept
File lines are stated in Mitigation Steps.

## Tools Used
Manual review 

## Recommended Mitigation Steps

#### In function calculateWithdrawOneTokenDY (L406)
```
uint256 _pooledTokensLength = self.pooledTokens.length;
```
The value is used twice, so one SLOAD is saved.

#### In function _calculateRemoveLiquidity (L953)
```
uint256 _pooledTokensLength = self.pooledTokens.length;
```
The value is used twice, one of those as condition in a for loop, so at least _pooledTokensLength SLOADS are saved.

#### In function _feePerToken (L1080)
```
uint256 _pooledTokensLength = self.pooledTokens.length;
```
The value is used twice, so one SLOAD is saved.

#### In function swap (L1098)
```
IERC20 tokenFrom = self.pooledTokens[tokenIndexFrom];
```
The value is used 4 times, so 3 SLOADs are saved.
However, this causes a stack too deep error in line 1129. To mitigate this, replace lines 1129-1132 for:
```
uint256 dyAdminFee = dyFee.mul(self.adminFee).div(FEE_DENOMINATOR);
        dyAdminFee = dyAdminFee.div(self.tokenPrecisionMultipliers[tokenIndexTo]);
```

#### In function addLiquidity (L1163)
```
uint256 _pooledTokensLength = self.pooledTokens.length;
uint256 _lpTokenTotalSupply = self.lpToken.totalSupply();
```
_pooledTokensLength is used 4 times. 3 SLOADs saved.
_lpTokenTotalSupply is used 6 times, however the one in line 1266 is called after a mint() so it's not the same value and thus can't be replaced. 4 SLOADs saved.

#### In function _updateUserWithdrawFee (L1290)
```
uint256 _withdrawFee = self.defaultWithdrawFee;
```
The value is used 3 times, so 2 SLOADs are saved.

#### In function removeLiquidityImbalance (L1415)
```
uint256 _pooledTokensLength = self.pooledTokens.length;
```
The value is used 5 times, twice as a for condition, so at least 2 + _pooledTokensLength SLOADs are saved.

