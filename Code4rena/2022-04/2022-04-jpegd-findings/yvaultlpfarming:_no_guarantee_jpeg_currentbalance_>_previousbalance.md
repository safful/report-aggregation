## Tags

- bug
- 3 (High Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [yVaultLPFarming: No guarantee JPEG currentBalance > previousBalance](https://github.com/code-423n4/2022-04-jpegd-findings/issues/56) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/farming/yVaultLPFarming.sol#L169-L170


# Vulnerability details

## Details & Impact

yVault users participating in the farm have to trust that:

- `vault.balanceOfJPEG()`  returns the correct claimable JPEG amount by its strategy / strategies
- the strategy / strategies will send all claimable JPEG to the farm

Should either of these assumptions break, then it could be possibly be the case that `currentBalance` is less than `previousBalance`, causing deposits and crucially, withdrawals to fail due to subtraction overflow.

## Proof of Concept

For instance, 

- Farm migration occurs. A new farm is set in `yVault`, then `withdrawJPEG()` is called, which sends funds to the new farm. Users of the old farm would be unable to withdraw their deposits.

```jsx
it.only("will revert old farms' deposits and withdrawals if yVault migrates farm", async () => {
  // 0. setup
  await token.mint(owner.address, units(1000));
  await token.approve(yVault.address, units(1000));
  await yVault.depositAll();
  await yVault.approve(lpFarming.address, units(1000));
  // send some JPEG to strategy prior to deposit
  await jpeg.mint(strategy.address, units(100));
  // deposit twice, so that the second deposit will invoke _update()
  await lpFarming.deposit(units(250));
  await lpFarming.deposit(units(250));
	
  // 1. change farm and call withdrawJPEG()
  await yVault.setFarmingPool(user1.address);
  await yVault.withdrawJPEG();
	
  // deposit and withdrawal will fail
  await expect(lpFarming.deposit(units(500))).to.be.revertedWith('reverted with panic code 0x11 (Arithmetic operation underflowed or overflowed outside of an unchecked block)');
  await expect(lpFarming.withdraw(units(500))).to.be.revertedWith('reverted with panic code 0x11 (Arithmetic operation underflowed or overflowed outside of an unchecked block)');
});
```

- Strategy migration occurs, but JPEG funds held by the old strategy were not claimed, causing `vault.balanceOfJPEG()` to report a smaller amount than previously recorded
- `jpeg` could be accidentally included in the StrategyConfig, resulting in JPEG being converted to other assets
- A future implementation takes a fee on the `jpeg` to be claimed

## Recommended Mitigation Steps

A simple fix would be to `return` if `currentBalance ≤ previousBalance`. A full fix would properly handle potential shortfall.

```jsx
if (currentBalance <= previousBalance) return;
```

