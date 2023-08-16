## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Constants "chi" and "rate"](https://github.com/code-423n4/2021-05-yield-findings/issues/13) 

# Handle

gpersoon


# Vulnerability details

## Impact
Several implementations of the value of "chi" and "rate" are used, sometimes as constant and sometimes the direct value is used, see proof of concept below.
The risk is that if it is changed in one place if might not be changed in another place, leading to bugs.

## Proof of Concept
// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Wand.sol#L26
bytes6 public constant CHI = "chi";
bytes6 public constant RATE = "rate";

// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/FYToken.sol#L27
bytes32 constant CHI = "chi";

//https://github.com/code-423n4/2021-05-yield/blob/main/contracts/oracles/compound/CompoundMultiOracle.sol#L40
    function _peek(bytes6 base, bytes6 kind) private view returns (uint price, uint updateTime) {
      ...
        if (kind == "rate") rawPrice = CTokenInterface(source).borrowIndex();
        else if (kind == "chi") rawPrice = CTokenInterface(source).exchangeRateStored();

## Tools Used
grep

## Recommended Mitigation Steps
Define the constants for "chi" and "rate" on one location and include this where required.


