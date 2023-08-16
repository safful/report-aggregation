## Tags

- bug
- sponsor confirmed
- 2 (Med Risk)
- resolved

# [`depositAndFix` can be made to fail](https://github.com/code-423n4/2021-10-tempus-findings/issues/20) 

# Handle

cmichel


# Vulnerability details

There's a griefing attack where an attacker can make any user transaction for `TempusController.depositAndFix` fail.
In `_depositAndFix`, `swapAmount` many yield shares are swapped to principal where `swapAmount` is derived from the function arguments.
A final `assert(yieldShares.balanceOf(address(this)) == 0)` statement checks that the yield shares of the contract are zero after the swap.
This is only true if no other yield shares were already in the contract.

However, an attacker can frontrun this call and send the smallest unit of yield shares to the contract which then makes the original deposit-and-fix transaction fail.

## Impact
All `depositAndFix` calls can be made to fail and this function becomes unusable.

## Recommended Mitigation Steps
Remove the `assert` check.

