## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed
- valid

# [Lack of event emission after sensitive action ](https://github.com/code-423n4/2022-08-rigor-findings/issues/49) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/main/contracts/HomeFi.sol#L92
https://github.com/code-423n4/2022-08-rigor/blob/main/contracts/HomeFi.sol#L113


# Vulnerability details

## Impact
The initialize function of the HomeFi contract does not emit the AdminReplaced event after setting the value of the  _msgSender() to be the admin.

Consider emitting events after sensitive changes occur to facilitate tracking and notify off-chain clients following the contracts’ activity.

## Proof of Concept
 https://github.com/code-423n4/2022-08-rigor/blob/main/contracts/HomeFi.sol#L92
https://github.com/code-423n4/2022-08-rigor/blob/main/contracts/HomeFi.sol#L113

## Tools Used
vscode
## Recommended Mitigation Steps
add event