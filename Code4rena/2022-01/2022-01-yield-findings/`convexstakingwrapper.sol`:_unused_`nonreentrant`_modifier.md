## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [`ConvexStakingWrapper.sol`: unused `nonReentrant` modifier](https://github.com/code-423n4/2022-01-yield-findings/issues/74) 

# Handle

Dravee


# Vulnerability details

## Impact
No protection from reentrancy (besides the gas limit on safeTransfer).
Bad practice compared to the original `ConvexStakingWrapper` contract.

## Proof of Concept
The original `ConvexStakingWrapper` contract used the `nonReentrant` modifier on all functions using the `safeTransfer` or `safeTransferFrom` methods:
- `deposit`: https://github.com/convex-eth/platform/blob/main/contracts/contracts/wrappers/ConvexStakingWrapper.sol#L337
- `stake`: https://github.com/convex-eth/platform/blob/main/contracts/contracts/wrappers/ConvexStakingWrapper.sol#L352 
- `withdraw`: https://github.com/convex-eth/platform/blob/main/contracts/contracts/wrappers/ConvexStakingWrapper.sol#L367
- `withdrawAndUnwrap`: https://github.com/convex-eth/platform/blob/main/contracts/contracts/wrappers/ConvexStakingWrapper.sol#L381

As the current one in the Yield solution is an upgrade, it should follow the same good practices.

## Tools Used
VS Code

## Recommended Mitigation Steps
Use the `nonReentrant` modifier on external functions that end up calling `safeTransfer` or `safeTransferFrom` (`user_checkpoint()` and `getReward()`)

