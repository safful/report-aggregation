## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Prefix increments are cheaper than postfix increments](https://github.com/code-423n4/2022-01-yield-findings/issues/14) 

# Handle

robee


# Vulnerability details

Prefix increments are cheaper than postfix increments. 
Further more, using unchecked {++x} is even more gas efficient, and the gas saving accumulates every iteration and can make a real change
There is no risk of overflow caused by increamenting the iteration index in for loops (the `++i` in `for (uint256 i = 0; i < numIterations; ++i)`).
But increments perform overflow checks that are not necessary in this case.
These functions use not using prefix increments (`++x`) or not using the unchecked keyword: 

        change to prefix increment and unchecked: ConvexStakingWrapper.sol, i, 115
        change to prefix increment and unchecked: ConvexStakingWrapper.sol, u, 172
        change to prefix increment and unchecked: ConvexStakingWrapper.sol, u, 227
        change to prefix increment and unchecked: ConvexYieldWrapper.sol, i, 111
        change to prefix increment and unchecked: ConvexStakingWrapper.sol, i, 287
        change to prefix increment and unchecked: ConvexStakingWrapper.sol, i, 271
        change to prefix increment and unchecked: ConvexStakingWrapper.sol, i, 315
        change to prefix increment and unchecked: ConvexYieldWrapper.sol, i, 63

