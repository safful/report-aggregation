## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [For Loops Need Break Statements (UserManager.sol)](https://github.com/code-423n4/2021-10-union-findings/issues/27) 

# Handle

ye0lde


# Vulnerability details

## Impact

There is no need to keep iterating through a loop for the full length once the condition being searched for is met.  This will save gas.

## Proof of Concept

The loops are here:
https://github.com/code-423n4/2021-10-union/blob/4176c366986e6d1a6b3f6ec0079ba547b040ac0f/contracts/user/UserManager.sol#L436-L441
https://github.com/code-423n4/2021-10-union/blob/4176c366986e6d1a6b3f6ec0079ba547b040ac0f/contracts/user/UserManager.sol#L444-L449
https://github.com/code-423n4/2021-10-union/blob/4176c366986e6d1a6b3f6ec0079ba547b040ac0f/contracts/user/UserManager.sol#L479-L485
https://github.com/code-423n4/2021-10-union/blob/4176c366986e6d1a6b3f6ec0079ba547b040ac0f/contracts/user/UserManager.sol#L488-L495

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps

Add a "break" statement to the loops mentioned above.
Note also that there are unnecessary default value initializations of variables associated with the loops.

