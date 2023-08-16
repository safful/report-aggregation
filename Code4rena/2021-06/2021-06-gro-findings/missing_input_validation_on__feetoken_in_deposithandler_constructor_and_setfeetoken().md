## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Missing input validation on _feeToken in DepositHandler constructor and setFeeToken()](https://github.com/code-423n4/2021-06-gro-findings/issues/45) 

# Handle

0xRajeev


# Vulnerability details

## Impact

There is no input validation on _feeToken in constructor to check if it's referring to a valid index (only USDT=2 makes sense) in the stablecoins similar to the check in setFeeToken(), which cannot be done here because the controller variable is only set later in setDependencies(). Also, given that it is set to true and that only USDT has this capability, the constructor should really check if this value is 2 and nothing else.

Also, setFeeToken() should only allow an index of 2 for now.

Scenario: Incorrectly using a _feeToken value other than 2 will cause an unnecessary balance check because of the presumed transfer fees for that token which does not exist.

## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L56

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L68-L75


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Check for _feeToken == 2 in constructor or set+check it using setFeeToken() later. Given that it is only USDT which may have fees, consider hardcoding this assumption instead of making it flexible and leaving room for error, because this is not something that applies to DAI or USDC. The entire codebase currently assumes the presence of only these three tokens in the protocol anyway.

