## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [More readable constants](https://github.com/code-423n4/2021-04-basedloans-findings/issues/8) 

# Handle

gpersoon


# Vulnerability details

## Impact
Some constant values are difficult to read in one time because they have at lot of 0's.
Solidity allows _ to separate series of zero's

## Proof of Concept
.\Governance\Blo.sol:    uint public constant totalSupply = 100000000e18; // 100 million BLO
.\Governance\GovernorAlpha.sol:    function quorumVotes() public pure returns (uint) { return 4000000e18; } // 4,000,000 = 4% of BLO
.\Governance\GovernorAlpha.sol:    function proposalThreshold() public pure returns (uint) { return 1000000e18; } // 1,000,000 = 1% of BLO

## Tools Used
grep

## Recommended Mitigation Steps
Replace   1000000e18 with    1_000_000e18 
Replace   4000000e18 with    4_000_000e18 
Replace 100000000e18 with  100_000_000e18 

