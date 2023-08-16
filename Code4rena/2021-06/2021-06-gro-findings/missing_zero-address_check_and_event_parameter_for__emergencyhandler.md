## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Missing zero-address check and event parameter for _emergencyHandler](https://github.com/code-423n4/2021-06-gro-findings/issues/49) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Controller setWithdrawHandler() is missing a zero-address check and event parameter for _emergencyHandler which is the more critical (used rarely but in an emergency incident-response that is always time-critical ) of the two addresses.

Scenario: setWithdrawHandler() is accidentally called with _emergencyHandler = 0 address. Without a check or an event here, this error goes unnoticed (unless caught in the event from WithdrawHandler::setDependencies). There is an emergency triggered after which withdrawals are attempted via the emergencyHandler but they fail because of the zero address. The correct non-zero emergencyHandler has to be set again. Valuable time is lost and funds are lost.


## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L105-L110

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L129

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L158


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add zero-address check and event parameter for _emergencyHandler

