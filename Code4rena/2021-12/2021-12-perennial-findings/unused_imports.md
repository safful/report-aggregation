## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Unused imports](https://github.com/code-423n4/2021-12-perennial-findings/issues/1) 

# Handle

robee


# Vulnerability details

In the following files there are contract imports that aren't used. 
Import of unnecessary files costs deployment gas (and is a bad coding practice that is important to ignore). 
The following is a full list of all unused imports, we went through the whole code to find it :) <solidity file> <line number> <actual import line>: 

        Incentivizer.sol, line 3, import "@openzeppelin/contracts/utils/math/SafeCast.sol";
        Program.sol, line 4, import "../../utils/types/Token18.sol";
        IIncentivizer.sol, line 5, import "../product/types/position/Position.sol";
        ChainlinkOracle.sol, line 5, import "../utils/types/UFixed18.sol";
        AccountPosition.sol, line 4, import "../accumulator/Accumulator.sol";

