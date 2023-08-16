## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Multiple double storage reading _xp(self) function calls](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/191) 

# Handle

hyh


# Vulnerability details

## Impact

Gas overspending due to excessive storage reads and function calls

## Proof of Concept

SwapUtils's removeLiquidityImbalance does multiple _xp(self) calls, which can be saved to memory when balances don't change inbetween executions
https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L1415

## Recommended Mitigation Steps

Now:
uint256[] memory balances1 = self.balances;
v.preciseA = determineA(self, _xp(self));
v.d0 = getD(_xp(self), v.preciseA);
...
v.d1 = getD(_xp(self, balances1), determineA(self, _xp(self, balances1)));
...
v.d2 = getD(_xp(self, balances1), determineA(self, _xp(self, balances1)));

To be:
uint256[] memory balances1 = self.balances;
uint256[] memory tokenPM = self.tokenPrecisionMultipliers; // doesn't change, save and reuse
uint256[] memory xP = _xp(balances1, tokenPM); // We already copied self.balances, no need to reread storage
v.d0 = getD(xP, determineA(self, xP)); // v.preciseA isn't used elsewhere and can be dropped
...
xP = _xp(balances1, tokenPM); // balances1 was modified, recomputing
v.d1 = getD(xP, determineA(self, xP));
...
xP = _xp(balances1, tokenPM); // balances1 was modified, recomputing
v.d2 = getD(xP, determineA(self, xP));

