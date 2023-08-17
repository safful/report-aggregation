## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Funds may be stuck when `redeeming` for Illuminate](https://github.com/code-423n4/2022-06-illuminate-findings/issues/384) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/redeemer/Redeemer.sol#L120


# Vulnerability details

## Impact
Funds may be stuck when `redeeming` for Illuminate.

## Proof of Concept
Assuming the goal of calling `redeem` for Illuminate [here](https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/redeemer/Redeemer.sol#L116) is to redeem the Illuminate principal held by the lender or the redeemer, then there is an issue because the wrong [balance](https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/redeemer/Redeemer.sol#L120) is checked. So if no `msg.sender` has a positive balance funds will be lost.


Now assuming the goal of calling `redeem` for Illuminate [here](https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/redeemer/Redeemer.sol#L116) is for users to redeem their Illuminate principal and receive the underlying as suggested by this [comment](https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/redeemer/Redeemer.sol#L127), then the underlying is not sent back to users because `Safe.transferFrom(IERC20(u), lender, address(this), amount);` send the funds to the redeemer, not the user. 


## Recommended Mitigation Steps
Clarify the purpose of this function and fix the corresponding bug.

