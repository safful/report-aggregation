## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Prefix increments are cheaper than postfix increments](https://github.com/code-423n4/2021-12-sublime-findings/issues/22) 

# Handle

robee


# Vulnerability details

Prefix increments are cheaper than postfix increments. 
Further more, using unchecked {++x} is even more gas efficient, and the gas saving accumulates every iteration and can make a real change
There is no risk of overflow caused by increamenting the iteration index in for loops (the `++i` in `for (uint256 i = 0; i < numIterations; ++i)`).
But increments perform overflow checks that are not necessary in this case.
These functions use not using prefix increments (`++x`) or not using the unchecked keyword: 

        change to prefix increment and unchecked: CreditLine.sol, _index, 484
        change to prefix increment and unchecked: CreditLine.sol, _index, 661
        change to prefix increment and unchecked: CreditLine.sol, _index, 735
        change to prefix increment and unchecked: CreditLine.sol, index, 888
        change to prefix increment and unchecked: CreditLine.sol, index, 955
        change to prefix increment and unchecked: SavingsAccount.sol, i, 286
        change to prefix increment and unchecked: SavingsAccount.sol, i, 464



