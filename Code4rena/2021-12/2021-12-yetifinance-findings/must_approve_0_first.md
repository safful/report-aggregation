## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Must approve 0 first](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/18) 

# Handle

robee


# Vulnerability details

Some tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value.They must first be approved by zero and then the actual allowance must be approved. 
You don't first approve 0 in the following places in the codebase: 

        approve without approving 0 first WJLP.sol, 127, JLP.approve(address(_MasterChefJoe), _amount); 
        approve without approving 0 first EchidnaProxy.sol, 134, return yusdToken.approve(spender, amount); 
        approve without approving 0 first sYETIToken.sol, 229, yusdToken.approve(routerAddress, YUSDToSell); 

