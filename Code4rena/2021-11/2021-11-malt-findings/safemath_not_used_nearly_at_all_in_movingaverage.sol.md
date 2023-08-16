## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [SafeMath Not Used Nearly At All In MovingAverage.sol](https://github.com/code-423n4/2021-11-malt-findings/issues/21) 

# Handle

jayjonah8


# Vulnerability details

## Impact
In MovingAverage.sol the safeMath.sol library is imported but I counted at least 25 places in the file where it should be used (Nearly the entire file).  This can result in values wrapping around which has caused devastating effects on many protocols in the past.  These values directly effect the exchangeRate variable given in the stabilize() function in StabilizerNode.sol so they must be treated with care. 

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/MovingAverage.sol#L3

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L156

## Tools Used
Manual code review 

## Recommended Mitigation Steps
The MovingAverage.sol file should be completely reviewed making use of safeMath through out the entire file.

