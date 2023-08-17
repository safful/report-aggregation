## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[H-05] Not minting iPTs for lenders in several lend functions](https://github.com/code-423n4/2022-06-illuminate-findings/issues/391) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L247-L305
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L317-L367
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L192-L235


# Vulnerability details

## Impact
Using any of the `lend` function mentioned, will result in loss of funds to the lender - as the funds are transferred from them but no iPTs are sent back to them!
Basically making lending via these external PTs unusable. 

## Proof of Concept
There is no minting of iPTs to the lender (or at all) in the 2 `lend` functions below:
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L247-L305
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L317-L367

Corresponding to lending of (respectively):
swivel
element

Furthermore, in:
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L227-L234
Comment says "Purchase illuminate PTs directly to msg.sender", but this is not happening. sending yield PTs at best.

## Recommended Mitigation Steps
Mint the appropriate amount of iPTs to the lender - like in the rest of the lend functions.

