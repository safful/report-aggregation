## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Typos](https://github.com/code-423n4/2021-10-slingshot-findings/issues/71) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-slingshot/blob/9c0432cca2e43731d5a0ae9c151dacf7835b8719/contracts/Slingshot.sol#L57-L73

```solidity=57
/// @notice Executes multi-hop trades to get the best result
///         It's up to BE to whitelist tokens
/// @param fromToken Start token address
/// @param toToken Target token address
/// @param fromAmount The initial amount of fromToken to start trading with
/// @param trades Array of encoded trades that are atomically executed
/// @param finalAmountMin The minimum expected output after all trades have been executed
/// @param depricated to be removed
function executeTrades(
    address fromToken,
    address toToken,
    uint256 fromAmount,
    TradeFormat[] calldata trades,
    uint256 finalAmountMin,
    address depricated
) external nonReentrant payable {
    depricated;
```

`depricated` should be `deprecated`.


