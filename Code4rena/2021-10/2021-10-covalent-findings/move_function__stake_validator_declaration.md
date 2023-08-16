## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Move Function _stake Validator Declaration](https://github.com/code-423n4/2021-10-covalent-findings/issues/89) 

# Handle

ye0lde


# Vulnerability details

## Impact

The variable v is declared after the first use of its contents.  Moving the declaration before the first use will save gas.

## Proof of Concept

"v" is declared here:
https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L180

But "v"s contents (validators[validatorId]) is used here first:
https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L178

Move line 180 above line 178 and change line 178 to use "v".

I've run out of contest time to continue testing but I'd recommend looking through the following functions for how "validators[validatorId]" may be used more efficiently.
https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L272
https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L346


## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
See Proof of Concept

