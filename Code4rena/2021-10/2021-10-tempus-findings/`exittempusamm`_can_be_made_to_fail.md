## Tags

- bug
- sponsor confirmed
- 2 (Med Risk)
- resolved

# [`exitTempusAMM` can be made to fail](https://github.com/code-423n4/2021-10-tempus-findings/issues/21) 

# Handle

cmichel


# Vulnerability details

There's a griefing attack where an attacker can make any user transaction for `TempusController.exitTempusAMM` fail.
In `_exitTempusAMM`, the user exits their LP position and claims back yield and principal shares.
The LP amounts to redeem are determined by the function parameter `lpTokensAmount`.
A final `assert(tempusAMM.balanceOf(address(this)) == 0)` statement checks that the LP token amount of the contract is zero after the exit.
This is only true if no other LP shares were already in the contract.

However, an attacker can frontrun this call and send the smallest unit of LP shares to the contract which then makes the original deposit-and-fix transaction fail.

## Impact
All `exitTempusAMM` calls can be made to fail and this function becomes unusable.

## Recommended Mitigation Steps
Remove the `assert` check.

