## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Must approve 0 first](https://github.com/code-423n4/2021-12-maple-findings/issues/11) 

# Handle

robee


# Vulnerability details

Some tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value.They must first be approved by zero and then the actual allowance must be approved. 
You don't first approve 0 in the following places in the codebase: 






        approve without approving 0 first SushiswapStrategy.sol, 25,         ERC20Helper.approve(fundsAsset_, lender_, repaymentAmount);

        approve without approving 0 first SushiswapStrategy.sol, 54,         ERC20Helper.approve(collateralAsset_, ROUTER, swapAmount_);

        approve without approving 0 first UniswapV2Strategy.sol, 25,         ERC20Helper.approve(fundsAsset_, lender_, repaymentAmount);

        approve without approving 0 first UniswapV2Strategy.sol, 54,         ERC20Helper.approve(collateralAsset_, ROUTER, swapAmount_);


