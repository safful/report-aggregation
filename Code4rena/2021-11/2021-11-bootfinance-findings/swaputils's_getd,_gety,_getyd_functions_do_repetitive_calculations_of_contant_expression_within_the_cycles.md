## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [SwapUtils's getD, getY, getYD functions do repetitive calculations of contant expression within the cycles](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/207) 

# Handle

hyh


# Vulnerability details

## Impact

Gas overspending due to excessive operations

## Proof of Concept

getD, getY, getYD functions calculate mul(d).div(xp[i].mul(numTokens) within the token cycles
https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L538
https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L588
https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L861

d, numTokens are constant there, so the divisions are redundant.

## Recommended Mitigation Steps

Introduce (d / numTokens) variable and simplify the multiplication

Now:
uint256 c = d;
...
for (uint256 i = 0; i < numTokens; i++) {
		if (i != tokenIndex) {
				s = s.add(xp[i]);
				c = c.mul(d).div(xp[i].mul(numTokens));

To be:
uint256 c = d;
uint256 d_num = d.div(numTokens);
...
for (uint256 i = 0; i < numTokens; i++) {
		if (i != tokenIndex) {
				s = s.add(xp[i]);
				c = c.mul(d_num).div(xp[i]);

