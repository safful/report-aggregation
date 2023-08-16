## Tags

- bug
- G (Gas Optimization)
- sponsor acknowledged
- sponsor confirmed

# [MarketPlace.sol: Remove maturity from VaultTracker and ZcToken](https://github.com/code-423n4/2021-09-swivel-findings/issues/29) 

# Handle

itsmeSTYJ


# Vulnerability details

## Impact

You don't need to store maturity in `VaultTracker.sol` or `ZcToken.sol` because `mapping (address => mapping (uint256 => bool)) public mature;` should already cover it. This will help to remove unnecessary external calls and also reduce the number of maturity checks.

## Recommended Mitigation Steps

```jsx
// In MarketPlace.sol
function createMarket(
	  address u,
	  uint256 m,
	  address c,
	  string memory n,
	  string memory s,
	  uint8 d
) public onlyAdmin(admin) returns (bool) {
	  require(swivel != address(0), 'swivel contract address not set');
	  // TODO can we live with the factory pattern here both bytecode size wise and CREATE opcode cost wise?
		address zctAddr = address(new ZcToken(u, n, s, d));
	  address vAddr = address(new VaultTracker(c, swivel));
	  markets[u][m] = Market(c, zctAddr, vAddr);
	
	  emit Create(u, m, c, zctAddr, vAddr);
	
	  return true;
}
...
function matureMarket(address u, uint256 m) public returns (bool) {
		require(block.timestamp >= m, "maturity not reached");
	  require(!mature[u][m], 'market already matured');
	
	  // set the base maturity cToken exchange rate at maturity to the current cToken exchange rate
	  uint256 currentExchangeRate = CErc20(markets[u][m].cTokenAddr).exchangeRateCurrent();
	  maturityRate[u][m] = currentExchangeRate;
	  // set the maturity state to true (for zcb market)
	  mature[u][m] = true;
	
	  // set vault "matured" to true
	  require(VaultTracker(markets[u][m].vaultAddr).matureVault(), 'maturity not reached');
	
	  emit Mature(u, m, block.timestamp, currentExchangeRate);
	
	  return true;
}
```

```jsx

// In VaultTracker.sol
...
// uint256 public immutable maturity; // deleted this
...
constructor(address c, address s) {
	  admin = msg.sender;
	  cTokenAddr = c;
	  swivel = s;
}
...
function matureVault() external onlyAdmin(admin) returns (bool) {
	  matured = true;
	  maturityRate = CErc20(cTokenAddr).exchangeRateCurrent();
	  return true;
}
```

```jsx
// uint256 public immutable maturity; // deleted this
...
constructor(address u, string memory n, string memory s, uint8 d) Erc2612(n, s, d) {
	  admin = msg.sender;
	  underlying = u;
}
```

