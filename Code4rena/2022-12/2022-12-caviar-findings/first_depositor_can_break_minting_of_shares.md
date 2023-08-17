## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-03

# [First depositor can break minting of shares](https://github.com/code-423n4/2022-12-caviar-findings/issues/442) 

# Lines of code

https://github.com/code-423n4/2022-12-caviar/blob/0212f9dc3b6a418803dbfacda0e340e059b8aae2/src/Pair.sol#L63


# Vulnerability details

## Impact
The attack vector and impact is the same as [TOB-YEARN-003](https://github.com/yearn/yearn-security/blob/master/audits/20210719_ToB_yearn_vaultsv2/ToB_-_Yearn_Vault_v_2_Smart_Contracts_Audit_Report.pdf), where users may not receive shares in exchange for their deposits if the total asset amount has been manipulated through a large “donation”.

## Proof of Concept
In `Pair.add()`, the amount of LP token minted is calculated as
```solidity
function addQuote(uint256 baseTokenAmount, uint256 fractionalTokenAmount) public view returns (uint256) {
    uint256 lpTokenSupply = lpToken.totalSupply();
    if (lpTokenSupply > 0) {
        // calculate amount of lp tokens as a fraction of existing reserves
        uint256 baseTokenShare = (baseTokenAmount * lpTokenSupply) / baseTokenReserves();
        uint256 fractionalTokenShare = (fractionalTokenAmount * lpTokenSupply) / fractionalTokenReserves();
        return Math.min(baseTokenShare, fractionalTokenShare);
    } else {
        // if there is no liquidity then init
        return Math.sqrt(baseTokenAmount * fractionalTokenAmount);
    }
}
```

An attacker can exploit using these steps
1. Create and add `1 wei baseToken - 1 wei quoteToken` to the pair. At this moment, attacker is minted `1 wei LP token` because `sqrt(1 * 1) = 1`
2. Transfer large amount of `baseToken` and `quoteToken` directly to the pair, such as `1e9 baseToken - 1e9 quoteToken`. Since no new LP token is minted, `1 wei LP token` worths `1e9 baseToken - 1e9 quoteToken`.
3. Normal users add liquidity to pool will receive `0` LP token if they add less than `1e9` token because of rounding division.
```solidity
baseTokenShare = (X * 1) / 1e9;
fractionalTokenShare = (Y * 1) / 1e9;
```


## Tools Used
Manual Review

## Recommended Mitigation Steps
- [Uniswap V2 solved this problem by sending the first 1000 LP tokens to the zero address](https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol#L119-L124). The same can be done in this case i.e. when `lpTokenSupply == 0`, send the first min liquidity LP tokens to the zero address to enable share dilution.
- In `add()`, ensure the number of LP tokens to be minted is non-zero:
```solidity
require(lpTokenAmount != 0, "No LP minted");
```

