## Tags

- bug
- sponsor confirmed
- 0 (Non-critical)
- resolved

# [lack of require message](https://github.com/code-423n4/2021-10-ambire-findings/issues/53) 

# Handle

JMukesh


# Vulnerability details

## Impact
require message give the idea what was the cause of failure , so its the best practise to  add message in require()

## Proof of Concept
https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L218

## Tools Used
manual reveiw

## Recommended Mitigation Steps
add message in require()

