## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Should include non-existing contract check](https://github.com/code-423n4/2021-11-malt-findings/issues/9) 

# Handle

jayjonah8


# Vulnerability details

## Impact
The executeTransaction() function in Timelock.sol does not include a check if the contract being called actually exists. The extcodesize is not used when using .call on addresses directly as per the solidity docs.  This is important because the EVM allows calls to a non-existing contract to always succeed. 

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Timelock.sol#L191

solidity docs: https://docs.soliditylang.org/en/v0.8.10/units-and-global-variables.html#address-related

"Due to the fact that the EVM considers a call to a non-existing contract to always succeed, Solidity includes an extra check using the extcodesize opcode when performing external calls. This ensures that the contract that is about to be called either actually exists (it contains code) or an exception is raised.
The low-level calls which operate on addresses rather than contract instances (i.e. .call(), .delegatecall(), .staticcall(), .send() and .transfer()) do not include this check, which makes them cheaper in terms of gas but also less safe."


## Tools Used
Manual code review

## Recommended Mitigation Steps
A check should be included to make sure the contract being called actually exists to avoid making possible errors in the executeTransaction() function

