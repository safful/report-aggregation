## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [VaultTracker.sol: init sVault.exchangeRate in constructor](https://github.com/code-423n4/2021-09-swivel-findings/issues/31) 

# Handle

itsmeSTYJ


# Vulnerability details

## Impact

Gas optimisation. In the function `transferNotionalFee()`,  `sVault.exchangeRate` is only 0 for the very first time this function is called so the if check to see if `sVault.exchangeRate != 0` is only used once to handle this edge case.

It makes more sense to set the exchangeRate when the vault is created and remove these if conditions.

## Recommended Mitigation Steps

```jsx
constructor(uint256 m, address c, address s) {
	  admin = msg.sender;
	  maturity = m;
	  cTokenAddr = c;
	  swivel = s;
		uint256 exchangeRate = CErc20(cTokenAddr).exchangeRateCurrent();
		vault[swivel] = Vault({
	      notional: 0,
	      redeemable: 0,
	      exchangeRate: exchangeRate
	  });
}
...
function transferNotionalFee(address f, uint256 a) external onlyAdmin(admin) returns(bool) {
    Vault memory oVault = vaults[f];
    Vault memory sVault = vaults[swivel];

    // remove notional from its owner
    oVault.notional -= a;

    uint256 exchangeRate = CErc20(cTokenAddr).exchangeRateCurrent();
    uint256 yield;
    uint256 interest;

    // check if exchangeRate has been stored already this block. If not, calculate marginal interest + store exchangeRate
    if (sVault.exchangeRate != exchangeRate) {
	      // if market has matured, calculate marginal interest between the maturity rate and previous position exchange rate
	      // otherwise, calculate marginal exchange rate between current and previous exchange rate.
	      if (matured) { 
		        // calculate marginal interest
	          yield = ((maturityRate * 1e26) / sVault.exchangeRate) - 1e26;
	      } else {
	          yield = ((exchangeRate * 1e26) / sVault.exchangeRate) - 1e26;
	      }
	
	      interest = (yield * sVault.notional) / 1e26;
	      sVault.redeemable += interest;
		    sVault.exchangeRate = exchangeRate;
    }

    // add notional to swivel's vault
    sVault.notional += a;

    // store the adjusted vaults
    vaults[swivel] = sVault;
    vaults[f] = oVault;
    return true;
  }
```

