## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Not verified function inputs of public / external functions](https://github.com/code-423n4/2021-12-perennial-findings/issues/11) 

# Handle

robee


# Vulnerability details

Not verified address arguments of external/public functions is a low risk issue. 
It's less severe for onlyOwner methods but for any other method it's crucial since the default address is 0.

        Argument account of Collateral.depositTo is not verified to be != 0
        Argument account of Collateral.withdrawTo is not verified to be != 0
        Argument account of Collateral.liquidate is not verified to be != 0
        Argument account of Collateral.settleAccount is not verified to be != 0
        Argument account of Collateral.collateral is not verified to be != 0
        Argument account of Collateral.liquidatable is not verified to be != 0
        Argument account of Collateral.liquidatableNext is not verified to be != 0
        Argument treasury_ of Factory.initialize is not verified to be != 0
        Argument controllerTreasury of Factory.createController is not verified to be != 0
        Argument newPauser of Factory.updatePauser is not verified to be != 0
        Argument account of Incentivizer.syncAccount is not verified to be != 0
        Argument account of Incentivizer.unclaimed is not verified to be != 0
        Argument account of Incentivizer.latestVersion is not verified to be != 0
        Argument account of Incentivizer.settled is not verified to be != 0
        Argument account of Product.settleAccount is not verified to be != 0
        Argument account of Product.closeAll is not verified to be != 0
        Argument account of Product.maintenance is not verified to be != 0
        Argument account of Product.maintenanceNext is not verified to be != 0
        Argument account of Product.isClosed is not verified to be != 0
        Argument account of Product.isLiquidating is not verified to be != 0
        Argument account of Product.position is not verified to be != 0
        Argument account of Product.pre is not verified to be != 0
        Argument account of Product.latestVersion is not verified to be != 0
        Argument recipient of MockToken18.push is not verified to be != 0
        Argument recipient of MockToken18.push is not verified to be != 0
        Argument benefactor of MockToken18.pull is not verified to be != 0
        Argument benefactor of MockToken18.pullTo is not verified to be != 0
        Argument recipient of MockToken18.pullTo is not verified to be != 0
        Argument account of MockToken18.balanceOf is not verified to be != 0
        Argument newOwner of UOwnable.transferOwnership is not verified to be != 0


