## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Incorrect Info in Comment in Alchemist.sol](https://github.com/code-423n4/2021-11-yaxis-findings/issues/5) 

# Handle

TimmyToes


# Vulnerability details

## Impact
Developers wishing to interact with yAxis will find it harder to do so.

## Proof of Concept
Lines 157-8 of Alchemist.sol
   /// @dev A mapping of all of the user CDPs. If a user wishes to have multiple CDPs they will have to either
    /// create a new address or set up a proxy contract that interfaces with this contract.
A proxy contract is not an option as most of the functions in the Alchemist contract have a noContractAllowed 
modifier.

## Recommended Mitigation Steps
Edit the comment to remove the proxy suggestion.


