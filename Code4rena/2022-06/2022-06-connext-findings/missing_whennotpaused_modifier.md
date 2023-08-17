## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Missing whenNotPaused modifier](https://github.com/code-423n4/2022-06-connext-findings/issues/175) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/facets/StableSwapFacet.sol#L279-L286


# Vulnerability details

## Impact
In `StableSwapFacet.sol`, two swapping functions contain the `whenNotPaused` modifier while `swapExactOut()` and `addSwapLiquidity()` do not. All functions to swap and add liquidity should contain the same modifiers to stop transactions while paused. 

## Proof of Concept
***Example with modifier***
```
  function swapExact(
    bytes32 canonicalId,
    uint256 amountIn,
    address assetIn,
    address assetOut,
    uint256 minAmountOut,
    uint256 deadline
  ) external payable nonReentrant deadlineCheck(deadline) whenNotPaused returns (uint256) {
```

***Examples without modifier***
```
  function swapExactOut(
    bytes32 canonicalId,
    uint256 amountOut,
    address assetIn,
    address assetOut,
    uint256 maxAmountIn,
    uint256 deadline
  ) external payable nonReentrant deadlineCheck(deadline) returns (uint256) {
```

and

```
  function addSwapLiquidity(
    bytes32 canonicalId,
    uint256[] calldata amounts,
    uint256 minToMint,
    uint256 deadline
  ) external nonReentrant deadlineCheck(deadline) returns (uint256) {
    return s.swapStorages[canonicalId].addLiquidity(amounts, minToMint);
  }
```

## Tools Used
Manual review.

## Recommended Mitigation Steps
Add the `whenNotPaused` modifier to all functions that perform swaps or liquidity additions.

