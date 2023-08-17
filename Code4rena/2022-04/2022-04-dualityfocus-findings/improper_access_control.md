## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Improper Access Control](https://github.com/code-423n4/2022-04-dualityfocus-findings/issues/25) 

# Lines of code

https://github.com/code-423n4/2022-04-dualityfocus/blob/f21ef7708c9335ee1996142e2581cb8714a525c9/contracts/compound_rari_fork/CToken.sol#L1641


# Vulnerability details

## Impact

In the referenced code this line,  `require(msg.sender != admin, "caller not admin");` is meant to prevent non-admins from calling the function however it instead prevents admins from calling the function and allows anyone else to. This could lead to defacing the token i.e changing the name to something offensive like Shit Token, Poo Coin, etc.

## Recommended Mitigation Steps

Adjust the require statement to reflect it's intended function i.e ` require(msg.sender == admin, "caller not admin");`

