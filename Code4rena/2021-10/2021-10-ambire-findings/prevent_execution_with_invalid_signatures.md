## Tags

- bug
- duplicate
- sponsor confirmed
- disagree with severity
- 3 (High Risk)
- resolved

# [Prevent execution with invalid signatures](https://github.com/code-423n4/2021-10-ambire-findings/issues/13) 

# Handle

gpersoon


# Vulnerability details

## Impact
Suppose one of the supplied addrs[i] to the constructor of Identity.sol happens to be 0 ( by accident).

In that case: privileges[0] = 1

Now suppose you call execute() with an invalid signature, then recoverAddrImpl will return a value of 0 and thus signer=0.
If you then check "privileges[signer] !=0"  this will be true and anyone can perform any transaction.

This is clearly an unwanted situation.

## Proof of Concept
https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/Identity.sol#L23-L30

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/Identity.sol#L97-L98

## Tools Used

## Recommended Mitigation Steps
In the constructor of Identity.sol, add in the for loop the following:
require (addrs[i] !=0,"Zero not allowed");

