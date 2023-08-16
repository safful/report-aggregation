## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [External call can be done later to save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/236) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L1362-L1375

```solidity=1362
function removeLiquidityOneToken(
    Swap storage self,
    uint256 tokenAmount,
    uint8 tokenIndex,
    uint256 minAmount
) external returns (uint256) {
    uint256 totalSupply = self.lpToken.totalSupply();
    uint256 numTokens = self.pooledTokens.length;
    require(
        tokenAmount <= self.lpToken.balanceOf(msg.sender),
        ">LP.balanceOf"
    );
    require(tokenIndex < numTokens, "Token not found");

```

The external call to get the `totalSupply` of the `lpToken` can be done later to avoid unnecessary code execution when the check of `tokenAmount` and `tokenIndex` does not pass.

