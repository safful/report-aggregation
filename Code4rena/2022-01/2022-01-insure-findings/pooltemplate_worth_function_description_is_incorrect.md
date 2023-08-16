## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [PoolTemplate worth function description is incorrect](https://github.com/code-423n4/2022-01-insure-findings/issues/189) 

# Handle

hyh


# Vulnerability details

## Impact

Underlying and index tokens are mixed up in the worth() function description, making code and its description conflicting

## Proof of Concept

Worth() computes how many iTokens correspond to given amount of underlying. The description says otherwise, mixing them up:

https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L794-798



## Recommended Mitigation Steps

Fix the description to say that ‘_value' is the amount of underlying, while the '_amount' is the corresponding output quantity of iTokens


