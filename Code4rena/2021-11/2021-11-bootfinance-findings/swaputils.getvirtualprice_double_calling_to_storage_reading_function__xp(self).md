## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [SwapUtils.getVirtualPrice double calling to storage reading function _xp(self)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/193) 

# Handle

hyh


# Vulnerability details

## Impact

Gas overspending due to excessive storage reads

## Proof of Concept

SwapUtils's getVirtualPrice repetitively calls _xp(self), which reads storage
https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L705

## Recommended Mitigation Steps

Now:
uint256 a = determineA(self, _xp(self));
uint256 d = getD(_xp(self), a);

To be:
uint256[] memory xP = _xp(self.balances, self.tokenPrecisionMultipliers);
uint256 d = getD(xP, determineA(self, xP));

