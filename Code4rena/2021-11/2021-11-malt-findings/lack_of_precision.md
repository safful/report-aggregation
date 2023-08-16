## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Lack of precision](https://github.com/code-423n4/2021-11-malt-findings/issues/168) 

# Handle

robee


# Vulnerability details

This issue is about arithmetic computation that could have been done more percise. 
The following are places in the codebase in which you multiplied after the divisions. 
Doing the multiplications at start lead to more accurate calculations. 
This is a list of places in the code that this appears (Solidity file, line number, actual line): 

        DAO.sol, 105,   /* Internal methods */

        UniswapHandler.sol, 265,         buyBase.div(priceTarget).mul(buyBase).mul(997)

        RewardDistributor.sol, 113,   /* PUBLIC VIEW FUNCTIONS */

        RewardDistributor.sol, 118,   /* INTERNAL VIEW FUNCTIONS */

        RewardDistributor.sol, 129,   /* INTERNAL FUNCTIONS */


