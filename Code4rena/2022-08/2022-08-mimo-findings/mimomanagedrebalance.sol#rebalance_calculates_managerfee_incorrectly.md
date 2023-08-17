## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [MIMOManagedRebalance.sol#rebalance calculates managerFee incorrectly](https://github.com/code-423n4/2022-08-mimo-findings/issues/34) 

# Lines of code

https://github.com/code-423n4/2022-08-mimo/blob/eb1a5016b69f72bc1e4fd3600a65e908bd228f13/contracts/actions/managed/MIMOManagedRebalance.sol#L50-L80


# Vulnerability details

## Impact
Inconsistent manager fees could lead to lack of incentivization to rebalance and unexpected liquidation.

## Proof of Concept

    uint256 managerFee = managedVault.fixedFee + flData.amount.wadMul(managedVault.varFee);

    IERC20(a.stablex()).safeTransfer(managedVault.manager, managerFee);

The variable portion of the fee is calculated using the amount of the flashloan but pays out in PAR. This is problematic because the value of the flashloan asset is constantly fluctuating in value against PAR. This results in an unpredictable fee for both the user and the manager. If the asset drops in price then the user will pay more than they intended. If the asset increases in price then the fee may not be enough to incentivize the manager to call them. The purpose of the managed rebalance is limit user interaction. If the manager isn't incentivized to call the vault then the user may be unexpectedly liquidated, resulting in loss of user funds.
  
## Tools Used

## Recommended Mitigation Steps

varFee should be calculated against the PAR of the rebalance like it is in MIMOAutoRebalance.sol:

    IPriceFeed priceFeed = a.priceFeed();
    address fromCollateral = vaultsData.vaultCollateralType(rbData.vaultId);

    uint256 rebalanceValue = priceFeed.convertFrom(fromCollateral, flData.amount);
    uint256 managerFee = managedVault.fixedFee + rebalanceValue.wadMul(managedVault.varFee);