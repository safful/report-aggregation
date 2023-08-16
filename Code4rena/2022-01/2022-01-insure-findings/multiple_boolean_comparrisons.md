## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Multiple boolean comparrisons](https://github.com/code-423n4/2022-01-insure-findings/issues/194) 

# Handle

loop


# Vulnerability details

When checking boolean values in a require or if statement it's an unnecessary operation to compare them to `true`, as it's already checked whether the condition is `true`. For comparison to `false`, it is cheaper to use the `!` operator rather than compare the value.

## Proof of Concept
Lines where boolean comparison is used:
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L99
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L131
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L161
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L176
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/CDSTemplate.sol#L205
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Factory.sol#L122
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Factory.sol#L142
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Factory.sol#L166-L169
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Factory.sol#L178-L179
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Factory.sol#L204
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L132
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L165
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L217
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L365
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L464
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L184
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L234
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L260
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L354
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L388
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L491
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L550
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L612
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L664

## Recommended Mitigation Steps
Remove the `== true` part from boolean comparisons and change `_variableName == false` to `!_variableName` to save some gas.

