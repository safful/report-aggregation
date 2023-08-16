## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- sponsor disputed

# [VaultTracker.sol: Gas optimisation for addNotional](https://github.com/code-423n4/2021-09-swivel-findings/issues/28) 

# Handle

itsmeSTYJ


# Vulnerability details

## Impact

Gas op. Since vlt.notional has to be updated in both branches of the if check, you can take vlt.notional out of both branches and skip the else check.

## Recommended Mitigation Steps

```jsx
function addNotional(address o, uint256 a) public onlyAdmin(admin) returns (bool) {
		uint256 exchangeRate = CErc20(cTokenAddr).exchangeRateCurrent();
		Vault memory vlt = vaults[o];
		
		if (vlt.notional > 0) {
			  uint256 yield;
			  uint256 interest;
			
			  // if market has matured, calculate marginal interest between the maturity rate and previous position exchange rate
			  // otherwise, calculate marginal exchange rate between current and previous exchange rate.
			  if (matured) { // Calculate marginal interest
				    yield = ((maturityRate * 1e26) / vlt.exchangeRate) - 1e26;
			  } else {
				    yield = ((exchangeRate * 1e26) / vlt.exchangeRate) - 1e26;
			  }
			
			  interest = (yield * vlt.notional) / 1e26;
			  // add interest and amount to position, reset cToken exchange rate
			  vlt.redeemable += interest;			 
		}
		
		vlt.notional += a;
		vlt.exchangeRate = exchangeRate;
		vaults[o] = vlt;
		
		return true;
}
```

