## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Variables that can be converted into immutables](https://github.com/code-423n4/2021-06-tracer-findings/issues/24) 

# Handle

hrkrshnn


# Vulnerability details

## Impact

Several variables can be converted into immutables. Doing so can save 2100 gas for storage reads when the slot is warm and 100 otherwise.

```
Notice: Variable declaration can be converted into an immutable.
  --> @openzeppelin/contracts/token/ERC20/ERC20.sol:38:5:
   |
38 |     uint256 private _totalSupply;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Insurance.sol:18:5:
   |
18 |     ITracerPerpetualsFactory public perpsFactory;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Insurance.sol:20:5:
   |
20 |     address public collateralAsset; // Address of collateral asset
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Insurance.sol:23:5:
   |
23 |     address public token; // token representation of a users holding in the pool
   |     ^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Insurance.sol:25:5:
   |
25 |     ITracerPerpetualSwaps public tracer; // Tracer associated with Insurance Pool
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Liquidation.sol:28:5:
   |
28 |     IPricing public pricing;
   |     ^^^^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Liquidation.sol:29:5:
   |
29 |     ITracerPerpetualSwaps public tracer;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Liquidation.sol:30:5:
   |
30 |     address public insuranceContract;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Liquidation.sol:31:5:
   |
31 |     address public fastGasOracle;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Pricing.sol:17:5:
   |
17 |     address public tracer;
   |     ^^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Pricing.sol:18:5:
   |
18 |     IInsurance public insurance;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^


Notice: Variable declaration can be converted into an immutable.
  --> contracts/Pricing.sol:19:5:
   |
19 |     IOracle public oracle;
   |     ^^^^^^^^^^^^^^^^^^^^^
```

## Proof of Concept

References to source file and line numbers can be seen above.

## Tools Used

A custom solidity compiler.

## Recommended Mitigation Steps

As explained above, these variables can be converted into immutables.

