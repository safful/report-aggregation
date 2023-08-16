## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Prefix increments are cheaper than postfix increments](https://github.com/code-423n4/2022-01-xdefi-findings/issues/9) 

# Handle

robee


# Vulnerability details

Prefix increments are cheaper than postfix increments. 
Further more, using unchecked {++x} is even more gas efficient, and the gas saving accumulates every iteration and can make a real change
There is no risk of overflow caused by increamenting the iteration index in for loops (the `++i` in `for (uint256 i = 0; i < numIterations; ++i)`).
But increments perform overflow checks that are not necessary in this case.
These functions use not using prefix increments (`++x`) or not using the unchecked keyword: 

        just change to unchecked: XDEFIDistribution.sol, i, 80
        just change to unchecked: XDEFIDistribution.sol, i, 212
        just change to unchecked: XDEFIDistribution.sol, i, 325
        just change to unchecked: XDEFIDistributionHelper.sol, i, 15
        just change to unchecked: XDEFIDistributionHelper.sol, i, 29
        just change to unchecked: XDEFIDistributionHelper.sol, i, 42


