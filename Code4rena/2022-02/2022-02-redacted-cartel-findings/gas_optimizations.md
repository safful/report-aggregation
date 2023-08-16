## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-02-redacted-cartel-findings/issues/99) 

##GasFindingsCartel
1--
-using storage to declare `b` struct
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/BribeVault.sol#L152
`b` just called twice in the `gotBribe`(struct data type consume more gas when it chaced and it depend on the size of the struct). Read from storage instead of caching it cost less gas
```
Bribe storage b = bribes[bribeIdentifier];
```

2--
-using at least pragma 0.8.4
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/BribeVault.sol#L2
The advantages of versions 0.8.4 over <0.8.0 are:
    1. Low level inliner : from 0.8.2, leads to cheaper runtime gas. Especially relevant when the contract has small functions. For example, OpenZeppelin libraries typically have a lot of small helper functions and if they are not inlined, they cost an additional 20 to 40 gas because of 2 extra jump instructions and additional stack operations needed for function calls.
    2. Optimizer improvements in packed structs: Before 0.8.3, storing packed structs, in some cases used an additional storage read operation. After EIP-2929, if the slot was already cold, this means unnecessary stack operations and extra deploy time costs. However, if the slot was already warm, this means additional cost of 100 gas alongside the same unnecessary stack operations and extra deploy time costs.
    3. Custom errors from 0.8.4, leads to cheaper deploy time cost and run time cost. Note: the run time cost is only relevant when the revert condition is met. In short, replace revert strings by custom errors.

3--
-using && cost more gas
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/BribeVault.sol#L262-L266
instead of using &&, using double require to validate can save gas
```
require(
            distributions.length == amounts.length,
            "Distributions, amounts, and fees must contain the same # of elements"
        );
require(
                distributions.length == fees.length,
            "Distributions, amounts, and fees must contain the same # of elements"
        );
```
4--
-using ++var for increment operation
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L109
using:
```
      ++reward.updateCount;
```   
is better method to do increment operation for gas opt

5--
-caching `Reward` to memory cost more gas
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L134
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L200
`reward` is merely called once at the `isRewardClaimed()` and twice at `_setClaimed()`. Its gas saving by just read it directly to the storage.
```
        Reward storage reward = rewards[_identifier];
```
