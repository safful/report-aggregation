## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Unconstrained fee](https://github.com/code-423n4/2022-02-concur-findings/issues/242) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/MasterChef.sol#L86-L101


# Vulnerability details

## Impact

Token fee in `MasterChef` can be set to more than 100%, (for example by accident) causing all `deposit` calls to fail due to underflow on subtraction when reward is lowered by the fee, thus breaking essential mechanics. Note that after the fee has been set to any value, it cannot be undone. A token cannot be removed, added, or added the second time. Thus, mistakenly (or deliberately, maliciously) added fee that is larger than 100% will make the contract impossible to recover from not being able to use the token.

## Tools Used

Manual analysis

## Recommended Mitigation Steps

On setting fee ensure that it is below a set maximum, which is set to no more than 100%.


