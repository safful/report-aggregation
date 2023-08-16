## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Variables that can be converted into immutables](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/110) 

# Handle

hrkrshnn


# Vulnerability details

## Variables that can be converted into immutables

``` txt
Warning: Variable declaration can be converted into an immutable.
  --> contracts/external/ERC20.sol:17:3:
   |
17 |   uint8 public decimals;
   |   ^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/PairFactory.sol:17:3:
   |
17 |   uint MAX_INT = 2**256 - 1;
   |   ^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/PairFactory.sol:22:3:
   |
22 |   address public lendingPairMaster;
   |   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/PairFactory.sol:23:3:
   |
23 |   address public lpTokenMaster;
   |   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/PairFactory.sol:24:3:
   |
24 |   IController public controller;
   |   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/RewardDistribution.sol:35:3:
   |
35 |   IPairFactory public factory;
   |   ^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/RewardDistribution.sol:36:3:
   |
36 |   IController  public controller;
   |   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/RewardDistribution.sol:37:3:
   |
37 |   IERC20  public rewardToken;
   |   ^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Instead of the expensive `sload`, to read from storage, these would be transformed into a cheap `push value`, when the variables are converted into immutable.

## Tools Used

A custom compiler.



