## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Several todos left in the code](https://github.com/code-423n4/2021-05-yield-findings/issues/9) 

# Handle

gpersoon


# Vulnerability details

## Impact
The code still has some todos, which should be resolved before production

## Proof of Concept
Ladle.sol:        weth.deposit{ value: ethTransferred }();   // TODO: Test gas savings using WETH10 `depositTo`
Ladle.sol:        weth.withdraw(ethTransferred);   // TODO: Test gas savings using WETH10 `withdrawTo`
Wand.sol:        cauldron.setRateOracle(assetId, IOracle(address(oracle))); // TODO: Consider adding a registry of chi oracles in cauldron as well
Wand.sol:        ); // TODO: Use a FYTokenFactory to make Wand deployable at 20000 runs
Wand.sol:            name,     // Derive from base and maturity, perhaps
Wand.sol:            symbol    // Derive from base and maturity, perhaps

## Tools Used
Grep

## Recommended Mitigation Steps
Check and fix or remove the todos


