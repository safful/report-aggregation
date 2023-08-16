## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [`approveUnstake` is unsafe ](https://github.com/code-423n4/2022-01-behodler-findings/issues/55) 

# Handle

CertoraInc


# Vulnerability details

Similar to ERC20.approve, `approveUnstake()` is unsafe due to the fact that it set the allowance to a fixed number and doesn't increase or decrease it.
Usually, the `ERC20.approve` doesn't get much attention because they leave it to the user to make sure his operation is safe, however here the user cannot do it because the `unstakeApproval` state variable is private and there is no getter for it.

In `ERC20.approve` users can simply check the allowance and change it in the same transaction and eliminate the risk, but here it's impossible.

## Impact
Users will not be able to change the allowance of the unstake without the risk of the frontrunning stealing like the classic `ERC20.approve` (there the risk can be removed).
This will cause users to not change allowance for users that they don't 100% trust which can be problematic

## Proof of Concept
The function that sets the allowance to a fixed number:
https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/Limbo.sol#L606-L612

The private map state variable that has no getter (in solidity state variables are automatically private unless declared otherwise)
https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/Limbo.sol#L288

## Tools Used
Manual code review

## Recommended Mitigation Steps
If you insist changing the allowance to a fixed number and not increase it or decrease it, at least make the allowance public so it can be checked before changing

