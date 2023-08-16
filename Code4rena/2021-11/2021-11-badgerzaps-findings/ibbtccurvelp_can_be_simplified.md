## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [ibbtcCurveLP can be simplified](https://github.com/code-423n4/2021-11-badgerzaps-findings/issues/21) 

# Handle

ye0lde


# Vulnerability details

## Impact

Removing unneeded branches and returns can reduce gas usage and improve code clarity.

## Proof of Concept

This code
https://github.com/Badger-Finance/ibbtc/blob/d8b95e8d145eb196ba20033267a9ba43a17be02c/contracts/Zap.sol#L309-L317
can be refactored to:

```
	if (bBtc <= max) {
		// pesimistically charge 0.5% on the withdrawal.
		// Actual fee might be lesser if the vault keeps keeps a buffer
		uint strategyFee = sett.mul(controller.strategies(pool.lpToken).withdrawalFee()).div(10000);
		lp = sett.sub(strategyFee).mul(pool.sett.getPricePerFullShare()).div(1e18);
		fee = fee.add(strategyFee);
	}
```

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
See POC

