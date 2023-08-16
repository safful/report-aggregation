## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [SingleStrategyController doesn't verify that new strategy uses the same base token](https://github.com/code-423n4/2022-03-prepo-findings/issues/62) 

# Lines of code

https://github.com/code-423n4/2022-03-prepo/blob/main/contracts/core/SingleStrategyController.sol#L51-L72
https://github.com/code-423n4/2022-03-prepo/blob/main/contracts/core/interfaces/IStrategy.sol#L52


# Vulnerability details

## Impact
When migrating from one strategy to another, the controller pulls out the funds of the old strategy and deposits them into the new one. But, it doesn't verify that both strategies use the same base token. If the new one uses a different base token, it won't "know" about the tokens it received on migration. It won't be able to deposit and transfer them. Effectively they would be lost.

The migration is done by the owner. So the owner must make a mistake and migrate to the wrong strategy by accident. In a basic protocol with 1 controller and a single active strategy managing that should be straightforward. There shouldn't be a real risk of that mistake happening. But, if you have multiple controllers running at the same time each with a different base token, it gets increasingly likelier. 

According to the `IStrategy` interface, there is a function to retrieve the strategy's base token: `getBaseToken()`. I'd recommend adding a check in the `migrate()` function to verify that the new strategy uses the correct base token to prevent this issue from being possible.

## Proof of Concept
https://github.com/code-423n4/2022-03-prepo/blob/main/contracts/core/SingleStrategyController.sol#L51-L72

https://github.com/code-423n4/2022-03-prepo/blob/main/contracts/core/interfaces/IStrategy.sol#L52

## Tools Used
none

## Recommended Mitigation Steps
Add  `require(_baseToken == _newStrategy.getBaseToken());` to the beginning of `migrate()`

