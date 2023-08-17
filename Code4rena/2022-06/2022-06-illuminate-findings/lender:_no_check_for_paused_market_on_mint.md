## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Lender: no check for paused market on mint](https://github.com/code-423n4/2022-06-illuminate-findings/issues/260) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L172


# Vulnerability details

Lender's `mint` function [does not check](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L172) whether the supplied market is paused.

## Impact
Even if a market is paused due to insolvency/bugs, an attacker can issue iPTs.
This renders the whole pause and insolvency protection mechanism ineffective.
See POC.

## Proof of Concept
Let's say market P has become insolvent, and Illuminate pauses that market, as it doesn't want to create further bad debt.
Let's say P's principal tokens's value has declined severely in the market because of the insolvency.
An attacker can buy many worthless P principal tokens for cheap, then call Lender and mint from them iPT.
The attacker is now owed underlying which belongs to the legitimate users. There won't be enough funds to repay everybody.

## Recommended Mitigation Steps
Check in `mint` that the market is not paused.

