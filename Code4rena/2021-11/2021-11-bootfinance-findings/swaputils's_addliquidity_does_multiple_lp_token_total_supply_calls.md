## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [SwapUtils's addLiquidity does multiple LP token total supply calls](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/197) 

# Handle

hyh


# Vulnerability details

## Impact

Gas overspending due to excessive external function calls

## Proof of Concept

SwapUtils's addLiquidity function calls LP token totalSupply() several times: 6 code occurrences, one is in cycle. The very last occurrency should be kept as it is, the first 5 of them should be replaced with memory variable as the supply changes only once when LP mint() is called at the end of the function.
https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L1163

## Recommended Mitigation Steps

Code update:

Now:
if (self.lpToken.totalSupply() != 0) { ...

To be:
uint256 lpTotalSupply = self.lpToken.totalSupply(); // storage read and function call
if (lpTotalSupply != 0) { ...

