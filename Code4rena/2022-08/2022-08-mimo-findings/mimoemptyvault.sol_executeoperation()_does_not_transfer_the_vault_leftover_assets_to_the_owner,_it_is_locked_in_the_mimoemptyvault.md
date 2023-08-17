## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [MIMOEmptyVault.sol executeOperation() does not transfer the Vault leftover assets to the owner, it is locked in the MIMOEmptyVault](https://github.com/code-423n4/2022-08-mimo-findings/issues/18) 

# Lines of code

https://github.com/code-423n4/2022-08-mimo/blob/eb1a5016b69f72bc1e4fd3600a65e908bd228f13/contracts/actions/MIMOEmptyVault.sol#L96-L100


# Vulnerability details

## Impact
MIMOEmptyVault.sol executeAction() is supposed to pay off the debt and return the leftover assets to the owner of the Vault
But In fact the emptyVault contract, after executing the executionOperation(), only pays back the flash loan, and does not transfer the leftover assets to the owner, and locked in the emptyVault contract

## Proof of Concept
```
  function executeOperation(
    address[] calldata assets,
    uint256[] calldata amounts,
    uint256[] calldata premiums,
    address initiator,
    bytes calldata params
  ) external override returns (bool) {

    ....
    ....

    require(flashloanRepayAmount <= vaultCollateral.balanceOf(address(this)), Errors.CANNOT_REPAY_FLASHLOAN);

    vaultCollateral.safeIncreaseAllowance(address(lendingPool), flashloanRepayAmount);

    //****Paid off the flash loan but did not transfer the remaining balance back to mimoProxy or owner ***//

    return true;
  }

```

Add logs to test case 

test/02_integration/MIMOEmtpyVault.test.ts

```
  it("should be able to empty vault with 1inch", async () => {
  ...
  ...
  ...
  ++++ console.log("before emptyVault balance:--->", (await wmatic.balanceOf(emptyVault.address)) + "");
    const tx = await mimoProxy.execute(emptyVault.address, MIMOProxyData);
    const receipt = await tx.wait(1);
  ++++ console.log("after emptyVault balance: --->", (await wmatic.balanceOf(emptyVault.address)) + "");  

```

print:
```
before emptyVault balance:---> 0
after emptyVault balance: ---> 44383268870065355782

```

## Tools Used

## Recommended Mitigation Steps

```
  function executeOperation(
    address[] calldata assets,
    uint256[] calldata amounts,
    uint256[] calldata premiums,
    address initiator,
    bytes calldata params
  ) external override returns (bool) {

    ....
    ....

    require(flashloanRepayAmount <= vaultCollateral.balanceOf(address(this)), Errors.CANNOT_REPAY_FLASHLOAN);

    vaultCollateral.safeIncreaseAllowance(address(lendingPool), flashloanRepayAmount);

    //****transfer the remaining balance back to mimoProxy or owner ***//
    ++++ vaultCollateral.safeTransfer(address(mimoProxy), vaultCollateral.balanceOf(address(this)) - flashloanRepayAmount);

    return true;
  }

```
