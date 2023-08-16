## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`USDMPegRecovery.provide()` Will Fail If There Is An Excess Of `usdm` Tokens](https://github.com/code-423n4/2022-02-concur-findings/issues/94) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/USDMPegRecovery.sol#L73-L82


# Vulnerability details

## Impact

The `provide` function does not take a `_steps` argument and will instead calculate `addingLiquidity` by truncating amounts under `step`. As a result, if there is an excess of `usdm` such that the truncated amount exceeds the contract's `pool3` truncated balance, then the function will revert due to insufficient `pool3` collateral.

This will prevent guardians from effectively providing liquidity whenever tokens are available. Consider the following example:
- The contract has `500000e18` `usdm` tokens and `250000e18` `pool3` tokens.
- `addingLiquidity` will be calculated as `500000e18 / 250000e18 * 250000e18`.
- The function will attempt to add `500000e18` `usdm` and `pool3` tokens in which there are insufficient `pool3` tokens in the contract. As a result, it will revert even though there is an abundance of tokens that satisfy the `step` amount.

## Proof of Concept

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/USDMPegRecovery.sol#L73-L82
```
function provide(uint256 _minimumLP) external onlyGuardian {
    require(usdm.balanceOf(address(this)) >= totalLiquidity.usdm, "<liquidity");
    // truncate amounts under step
    uint256 addingLiquidity = (usdm.balanceOf(address(this)) / step) * step;
    // match usdm : pool3 = 1 : 1
    uint256[2] memory amounts = [addingLiquidity, addingLiquidity];
    usdm.approve(address(usdm3crv), addingLiquidity);
    pool3.approve(address(usdm3crv), addingLiquidity);
    usdm3crv.add_liquidity(amounts, _minimumLP);
}
```

## Tools Used

Manual code review.
Discussions with Taek.

## Recommended Mitigation Steps

Consider modifying the `provide` function such that a `_steps` argument can be supplied. This will allow guardians to maximise the amount of liquidity provided to the Curve pool.

