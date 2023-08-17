## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Redeemer.redeem() for Element withdraws PT to wrong address.](https://github.com/code-423n4/2022-06-illuminate-findings/issues/182) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/92cbb0724e594ce025d6b6ed050d3548a38c264b/redeemer/Redeemer.sol#L187


# Vulnerability details

## Impact
Redeemer.redeem() for Element withdraws PT to wrong address.
This might cause a result of loss of PT.


## Proof of Concept
According to the ReadMe.md, Redeemer should transfer external principal tokens from Lender.sol to Redeemer.sol.
But it transfers to the "marketPlace" and it would lose the PT.


## Tools Used
Manual Review


## Recommended Mitigation Steps
Modify [IElementToken(principal).withdrawPrincipal(amount, marketPlace);](https://github.com/code-423n4/2022-06-illuminate/blob/92cbb0724e594ce025d6b6ed050d3548a38c264b/redeemer/Redeemer.sol#L187) like this.

```
IElementToken(principal).withdrawPrincipal(amount, address(this));
```

