## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Compare with 0 and 1 in a more efficient way](https://github.com/code-423n4/2021-10-ambire-findings/issues/15) 

# Handle

gpersoon


# Vulnerability details

## Impact
In the function setAddrPrivilege of Identity.sol the value of privileges[addr] is compare to 0 and 1 in the following way:
"if (privileges[addr] != bytes32(0) && privileges[addr] != bytes32(uint(1)))"

As 0 and 1 are adjacent, you could also check "uint(privileges[addr]) > 1". This saves a (small amount) of gas.

## Proof of Concept
https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/Identity.sol#L59

## Tools Used

## Recommended Mitigation Steps
replace
if (privileges[addr] != bytes32(0) && privileges[addr] != bytes32(uint(1))) ...
with
if (uint(privileges[addr]) > 1) ...

