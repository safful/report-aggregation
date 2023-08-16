## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Anyone can call release() in OLETokenLock.sol](https://github.com/code-423n4/2022-01-openleverage-findings/issues/56) 

# Handle

jayjonah8


# Vulnerability details

## Impact
In OLETokenLock.sol,  the release() function distributes all the allotted tokens to the beneficiaries but it can be called by anyone.  This should be an admin protected function as it's very important and deals with the transfer of tokens to beneficiaries which should not be accessed by simply anyone. 

## Proof of Concept
https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/OLETokenLock.sol#L39

## Tools Used
Manual code review 

## Recommended Mitigation Steps
OLETokenLock.sol should inherit the Adminable.sol contract and add require(msg.sender = admin, "Not Authorized"); to the release() function. 

