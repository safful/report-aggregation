## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Swivel: Taker is charged fees twice in exitVaultFillingVaultInitiate](https://github.com/code-423n4/2021-09-swivel-findings/issues/39) 

# Handle

itsmeSTYJ


# Vulnerability details

## Impact

Taker is charged fees twice in `exitVaultFillingVaultInitiate()` . Maker is transferring less than premiumFilled to taker and then taker is expected to pay fees i.e. taker's net balance is premiumFilled - 2*fee

## Recommended Mitigation Steps

```jsx
function exitVaultFillingVaultInitiate(Hash.Order calldata o, uint256 a, Sig.Components calldata c) internal {
    bytes32 hash = validOrderHash(o, c);

    require(a <= (o.principal - filled[hash]), 'taker amount > available volume');
    
    filled[hash] += a;
        
    uint256 premiumFilled = (((a * 1e18) / o.principal) * o.premium) / 1e18;
    uint256 fee = ((premiumFilled * 1e18) / fenominator[3]) / 1e18;

    Erc20 uToken = Erc20(o.underlying);
    // transfer premium from maker to sender
    uToken.transferFrom(o.maker, msg.sender, premiumFilled);

    // transfer fee in underlying to swivel from sender
    uToken.transferFrom(msg.sender, address(this), fee);

    // transfer <a> vault.notional (nTokens) from sender to maker
    require(MarketPlace(marketPlace).p2pVaultExchange(o.underlying, o.maturity, msg.sender, o.maker, a), 'vault exchange failed');

    emit Exit(o.key, hash, o.maker, o.vault, o.exit, msg.sender, a, premiumFilled);
}
```

