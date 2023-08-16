## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Variables that can be  converted into immutable](https://github.com/code-423n4/2021-07-spartan-findings/issues/73) 

# Handle

hrkrshnn


# Vulnerability details

## Variables that can be converted into immutable

``` txt
Warning: Variable declaration can be converted into an immutable.
  --> contracts/BondVault.sol:12:5:
   |
12 |     address public BASE;
   |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Dao.sol:16:5:
   |
16 |     address public BASE;
   |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Dao.sol:18:5:
   |
18 |     uint256 public secondsPerEra;   // Amount of seconds per era (Inherited from BASE contract; intended to be ~1 day)
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/DaoVault.sol:12:5:
   |
12 |     address public BASE;
   |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/DaoVault.sol:13:5:
   |
13 |     address public DEPLOYER;
   |     ^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Pool.sol:14:5:
   |
14 |     address public BASE;
   |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Pool.sol:15:5:
   |
15 |     address public TOKEN;
   |     ^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Pool.sol:16:5:
   |
16 |     address public DEPLOYER;
   |     ^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Pool.sol:19:5:
   |
19 |     uint8 public override decimals; uint256 public override totalSupply;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Pool.sol:27:5:
   |
27 |     uint public genesis; // Timestamp from when the pool was first deployed (For UI)
   |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
 --> contracts/Router.sol:9:5:
  |
9 |     address public BASE;
  |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Router.sol:10:5:
   |
10 |     address public WBNB;
   |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Router.sol:11:5:
   |
11 |     address public DEPLOYER;
   |     ^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
 --> contracts/Synth.sol:7:5:
  |
7 |     address public BASE;
  |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
 --> contracts/Synth.sol:8:5:
  |
8 |     address public LayerONE; // Underlying relevant layer1 token
  |     ^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
 --> contracts/Synth.sol:9:5:
  |
9 |     uint public genesis;
  |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Synth.sol:10:5:
   |
10 |     address public DEPLOYER;
   |     ^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Synth.sol:13:5:
   |
13 |     uint8 public override decimals; uint256 public override totalSupply;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Utils.sol:10:5:
   |
10 |     address public BASE;
   |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/Utils.sol:11:5:
   |
11 |     uint public one = 10**18;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/outside-scope/FallenSpartans.sol:10:5:
   |
10 |     address public SPARTA;
   |     ^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/outside-scope/FallenSpartans.sol:11:5:
   |
11 |     address public DEPLOYER;
   |     ^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/outside-scope/FallenSpartans.sol:12:5:
   |
12 |     uint256 public genesis;
   |     ^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
 --> contracts/outside-scope/Reserve.sol:8:5:
  |
8 |     address public BASE;
  |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/outside-scope/Sparta.sol:30:5:
   |
30 |     uint256 private _100m;
   |     ^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/outside-scope/Sparta.sol:31:5:
   |
31 |     uint256 public maxSupply;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/outside-scope/Sparta.sol:38:5:
   |
38 |     address public BASEv1;
   |     ^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
 --> contracts/poolFactory.sol:7:5:
  |
7 |     address public BASE;
  |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
 --> contracts/poolFactory.sol:8:5:
  |
8 |     address public WBNB;
  |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/poolFactory.sol:10:5:
   |
10 |     uint public curatedPoolSize;    // Max amount of pools that can be curated status
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
 --> contracts/synthFactory.sol:6:5:
  |
6 |     address public BASE;
  |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
 --> contracts/synthFactory.sol:7:5:
  |
7 |     address public WBNB;
  |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/synthVault.sol:14:5:
   |
14 |     address public BASE;
   |     ^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/synthVault.sol:15:5:
   |
15 |     address public DEPLOYER;
   |     ^^^^^^^^^^^^^^^^^^^^^^^


Warning: Variable declaration can be converted into an immutable.
  --> contracts/synthVault.sol:23:5:
   |
23 |     uint public genesis;                // Timestamp from when the synth was first deployed (For UI)
   |     ^^^^^^^^^^^^^^^^^^^

```


Instead of using an expensive `sload` operation, converting to immutable would make reading to cost just 3 gas.


## Tools Used

A custom compiler.

