## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Unused imports](https://github.com/code-423n4/2021-10-tracer-findings/issues/33) 

# Handle

pauliax


# Vulnerability details

## Impact
There are unused imports. They will increase the size of deployment with no real benefit. Consider removing unused imports to save some gas. 

Examples of such imports are: 

in contract PoolKeeper
import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV2V3Interface.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/proxy/Clones.sol";

imported twice:
import "../interfaces/IERC20DecimalsWrapper.sol";

in contract PoolCommitter
import "../interfaces/IOracleWrapper.sol";

## Recommended Mitigation Steps
Remove unnecessary imports.

