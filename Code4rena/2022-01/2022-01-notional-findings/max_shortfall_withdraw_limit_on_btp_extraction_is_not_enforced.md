## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [MAX_SHORTFALL_WITHDRAW limit on BTP extraction is not enforced](https://github.com/code-423n4/2022-01-notional-findings/issues/209) 

# Handle

gellej


# Vulnerability details

## Impact


The function `extractTokensForCollateralShortfall()` allows the owner of the sNote contract to withdraw up to 50% of the total amount of BPT. 

Presumably, this 50% limit is in place to prevent the owner from "rug-pulling" the sNote holders (or at least to give them a guarantee that their loss is limited to 50% of the underlying value). 

However, this limit is easily circumvented as the function can simply be called a second, third and fourth time, to withdraw almost all of the BPT. 

As the contract does not enforce this limit, the bug requires stakers to trust the governance to not withdraw more than 50% of the underlying collateral. This represents a higher risk for the stakers, which may  also result in a larger discount on sNote wrt its BPT collateral (this is why I classified the bug as medium risk - users may lose value - not from an exploit, but from the lack of enforcing the 50% rule)

# Proof of Concept

See above.
The code affected is here: https://github.com/code-423n4/2022-01-notional/blob/main/contracts/sNOTE.sol#L100



## Recommended Mitigation Steps

Rewrite the logic and enforce a limit during a time period - i.e. do not allow to withdraw over 50% _per week_ (or any time period that is longer than the cooldown period, so that users have time to withdraw their collateral)

