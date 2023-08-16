## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [The function `updateValset` does not have enough sanity checks](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/51) 

# Handle

hrkrshnn


# Vulnerability details

##  `updateValset` does not have enough sanity checks

In
[updateValset](https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/solidity/contracts/Gravity.sol#L224)
function, the current set of validators adds a new set.

It is missing the check that the combined power of all new validators is
above the `state_powerThreshold`. If this is false, then the contract is
effectively stuck. Consider adding an on-chain check for this.

It is also worth adding a that the size of the new validator check is
less than a certain number.

Here is a rough calculation explaining how 10000 validators (an extreme
example) is too much:

1.  Let us say that the new set of validators have the property that at
    least, say, `N` validators are needed to get the total threshold
    above `state_powerThreshold`.
2.  Since each validating signature requires a call to `ecrecover`,
    costing at least `3000` gas, the minimum gas needed for getting a
    proposal over `state_powerThreshold` would be `N * 3000`
3.  `N * 3000` cannot be more than the `block.gaslimit` Currently, this
    puts `N` to be less than `10000`

Another approach to solve the above potential problems is to do the
updating as a two step process:

1.  The current set of validators proposes a pending set of validators.
2.  And the pending set of validators need to do the transition to
    become the new set of validators. Going through the same threshold
    checks.

This guarantees that the new set of validators has enough power to pass
threshold and doesn't have gas limit issues in doing so.


