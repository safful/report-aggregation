## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [VaultTracker.sol: pass in exchangeRate as a variable to matureVault()](https://github.com/code-423n4/2021-09-swivel-findings/issues/30) 

# Handle

itsmeSTYJ


# Vulnerability details

## Impact

Since you are already querying the exchangeRate for the current block in `MarketPlace.matureMarket()` , might as well pass it along to `VaultTracker.sol` instead of querying it a second time.

## Recommended Mitigation Steps

```jsx
// In VaultTracker.sol
function matureVault(uint256 _maturityRate) external onlyAdmin(admin) returns (bool) {
	  require(!matured, 'already matured');
	  require(block.timestamp >= maturity, 'maturity has not been reached');
	  matured = true;
		maturityRate = _maturityRate;
	  return true;
}
```

```jsx
// In MarketPlace.sol
function matureMarket(address u, uint256 m) public returns (bool) {
	  require(!mature[u][m], 'market already matured');
	  require(block.timestamp >= ZcToken(markets[u][m].zcTokenAddr).maturity(), "maturity not reached");
	
	  // set the base maturity cToken exchange rate at maturity to the current cToken exchange rate
	  uint256 currentExchangeRate = CErc20(markets[u][m].cTokenAddr).exchangeRateCurrent();
	  maturityRate[u][m] = currentExchangeRate;
	  // set the maturity state to true (for zcb market)
	  mature[u][m] = true;
	
	  // set vault "matured" to true
		require(VaultTracker(markets[u][m].vaultAddr).matureVault(currentExchangeRate), 'maturity not reached');
	
	  emit Mature(u, m, block.timestamp, currentExchangeRate);
	
	  return true;
}
```

