## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Controller: Strategy migration will fail](https://github.com/code-423n4/2022-04-jpegd-findings/issues/57) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/vaults/yVault/Controller.sol#L95
https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/vaults/yVault/strategies/StrategyPUSDConvex.sol#L266


# Vulnerability details

## Details

The controller calls the `withdraw()` method to withdraw JPEGs from the contract, but the strategy might blacklist the JPEG asset, which is what the PUSDConvex strategy has done.

The migration would therefore revert.

## Proof of Concept

Insert this test into [`StrategyPUSDConvex.ts`](https://github.com/code-423n4/2022-04-jpegd/blob/main/tests/StrategyPUSDConvex.ts).

```jsx
it.only("will revert when attempting to migrate strategy", async () => {
  await controller.setVault(want.address, yVault.address);
  await expect(controller.setStrategy(want.address, strategy.address)).to.be.revertedWith("jpeg");
});
```

## Recommended Mitigation Steps

Replace `_current.withdraw(address(jpeg));` with `_current.withdrawJPEG(vaults[_token])`.

