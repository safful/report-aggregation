## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [Lack of event emission after critical initialize() functions](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/68) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Most contracts use initialize() functions instead of constructor given the delegatecall proxy pattern. While most of them emit an event in the critical initialize() functions to record the init parameters for off-chain monitoring and transparency reasons, Ticket.sol nor its base class ControlledToken.sol emit such an event in their initialize() functions.

Impact: These contracts are initialized but their critical init parameters (name, symbol, decimals and controller address) are not logged for any off-chain monitoring.

## Proof of Concept

See similar Medium-severity Finding M01 in OpenZeppelin’s audit of UMA protocol: https://blog.openzeppelin.com/uma-audit-phase-4/

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/Ticket.sol#L24-L37

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/ControlledToken.sol#L22-L36

Examples of event emission:
https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L239-L243

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/YieldSourcePrizePool.sol#L47


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Emit an initialised event in Ticket.sol and ControlledToken.sol logging their init parameters.

