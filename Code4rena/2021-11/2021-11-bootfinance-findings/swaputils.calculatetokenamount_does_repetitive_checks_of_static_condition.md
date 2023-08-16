## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [SwapUtils.calculateTokenAmount does repetitive checks of static condition](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/200) 

# Handle

hyh


# Vulnerability details

## Impact

Gas overspending due to excessive operations

## Proof of Concept

SwapUtils.calculateTokenAmount's 'deposit' bool variable is checked on each iteration, while one check is enough
https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L1031

## Recommended Mitigation Steps

It's recommended to separate the cycles:

Now:
for (uint256 i = 0; i < numTokens; i++) {
		if (deposit) {
				balances1[i] = balances1[i].add(amounts[i]);
		} else {
				balances1[i] = balances1[i].sub(
						amounts[i],
						"Cannot withdraw more than available"
				);
		}
}

To be:
if (deposit) {
		for (uint256 i = 0; i < numTokens; i++) {
			balances1[i] = balances1[i].add(amounts[i]);
		}
} else {
		for (uint256 i = 0; i < numTokens; i++) {
			balances1[i] = balances1[i].sub(
					amounts[i],
					"Cannot withdraw more than available"
			);
		}
}


