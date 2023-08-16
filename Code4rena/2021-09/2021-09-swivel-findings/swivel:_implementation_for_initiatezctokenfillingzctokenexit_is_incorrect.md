## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Swivel: implementation for initiateZcTokenFillingZcTokenExit is incorrect](https://github.com/code-423n4/2021-09-swivel-findings/issues/38) 

# Handle

itsmeSTYJ


# Vulnerability details

## Impact

In `initiateZcTokenFillingZcTokenExit()` , this comment `// transfer underlying tokens - the premium paid + fee in underlying to swivel (from sender)`  is incorrect because you are actually transferring the underlying tokens - premium paid to the maker (from sender) AND you have to pay fee separately to swivel.

initiateZCTokenFillingZcTokenExit means I want to sell my nTokens so that means `a` is the amount of principal I want to fill. Let's use a hypothetical example where I (taker) wants to fill 10 units of ZcTokenExit for maker.

1. I transfer 10 units of underlying to Swivel. The net balances are: me (-a), swivel (+a)
2. I transfer fee (in underlying) to Swivel. The net balances are: me (-a-fee), swivel (+a+fee) 
3. Swivel initiates my position, sends me the ZcToken and sends Maker the nTokens
4. Maker pays me premiumFilled for the nTokens. The net balances are: me (-a-fee+premiumsFilled), swivel (+a+fee), maker (-premiumsFilled)
5. Maker closes position. The net balances are: me (-a-fee+premiumsFilled), swivel (+fee), maker (-premiumsFilled+a)

So effectively, I (taker) should be paying a-premium to maker and fee to swivel.

## Recommended Mitigation Steps

```jsx
function initiateZcTokenFillingZcTokenExit(Hash.Order calldata o, uint256 a, Sig.Components calldata c) internal {
    bytes32 hash = validOrderHash(o, c);

    require(a <= o.principal - filled[hash]), 'taker amount > available volume'); // Note: you don't need to wrap these in brackets because if you look at the https://docs.soliditylang.org/en/latest/cheatsheet.html#order-of-precedence-of-operators, subtraction will always go before comparison 

    filled[hash] += a;

    uint256 premiumFilled = (((a * 1e18) / o.principal) * o.premium) / 1e18;
    uint256 fee = ((premiumFilled * 1e18) / fenominator[0]) / 1e18;

    // transfer underlying tokens - the premium paid in underlying to maker (from sender)
    Erc20(o.underlying).transferFrom(msg.sender, o.maker, a - premiumFilled);
		Erc20(o.underlying).transferFrom(msg.sender, swivel, fee);
    // transfer <a> zcTokens between users in marketplace
    require(MarketPlace(marketPlace).p2pZcTokenExchange(o.underlying, o.maturity, o.maker, msg.sender, a), 'zcToken exchange failed');
            
    emit Initiate(o.key, hash, o.maker, o.vault, o.exit, msg.sender, a, premiumFilled);
}
```

