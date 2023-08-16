## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [[WP-M0] `USDMPegRecovery.sol#provide()` Improper design/implementation make it often unable to add liquidity to the `usdm3crv` pool](https://github.com/code-423n4/2022-02-concur-findings/issues/191) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/02d286253cd5570d4e595527618366f77627cdaf/contracts/USDMPegRecovery.sol#L73-L82


# Vulnerability details

https://github.com/code-423n4/2022-02-concur/blob/02d286253cd5570d4e595527618366f77627cdaf/contracts/USDMPegRecovery.sol#L73-L82

```solidity
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

In the current implementation of `USDMPegRecovery.sol#provide()`, `addingLiquidity` is calculated solely based on `usdm` balance (truncate at a step of 250k), and it always uses the same amount of 3pool tokens to add_liquidity with.

Based on other functions of the contract, the balance of `usdm` can usually be more than the `pool3` balance, in that case, `usdm3crv.add_liquidity()` will fail.

### Impact

When the balance of `pool3` is less than `usdm` (which is can be a common scenario), funds cannot be added to the curve pool.

For example:

When the contract got 5M of USDM and 4.2M of `pool3` tokens, it won't be possible to call `provide()` and add liquidity to the `usdm3crv` pool, as there are not enough pool3 tokens to match the 5M of USDM yet.

We expect it to add liquidity with 4M of USDM and 4M of pool3 tokens in that case.

### Recommendation

Change to:

```solidity
function provide(uint256 _minimumLP) external onlyGuardian {
    require(usdm.balanceOf(address(this)) >= totalLiquidity.usdm, "<liquidity");
    uint256 tokenBalance = Math.min(usdm.balanceOf(address(this), pool3.balanceOf(address(this));
    // truncate amounts under step
    uint256 addingLiquidity = (tokenBalance / step) * step;
    // match usdm : pool3 = 1 : 1
    uint256[2] memory amounts = [addingLiquidity, addingLiquidity];
    usdm.approve(address(usdm3crv), addingLiquidity);
    pool3.approve(address(usdm3crv), addingLiquidity);
    usdm3crv.add_liquidity(amounts, _minimumLP);
}
```

