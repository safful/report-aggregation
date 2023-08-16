## Tags

- bug
- duplicate
- 0 (Non-critical)
- sponsor confirmed

# [Check if address is a contract](https://github.com/code-423n4/2021-08-notional-findings/issues/47) 

# Handle

pauliax


# Vulnerability details

## Impact
There are several places that check if the address is a contract or not, e.g.:
    uint256 codeSize;
    assembly {
        codeSize := extcodesize(operator)
    }
    // Sanity check to ensure that operator is a contract, not an EOA
    require(codeSize > 0, "Operator must be a contract");
First of all, I want you to be aware that this check can be easily bypassed. A contract does not have source code available during construction. This means that while the constructor is running, it can make calls to other contracts, but extcodesize for its address returns zero. In your case, currently, I do not see a real problem with that as you only use it as an additional check (no critical functionality) but I have a suggestion for you to extract this check to a separate library to make it more maintainable or use a library from OpenZeppelin that exposes this function: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L26


