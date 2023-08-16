## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Double check for "birthday" collision](https://github.com/code-423n4/2021-08-notional-findings/issues/4) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function getRouterImplementation of Router.sol checks the selectors of functions and calls the appropriate function.
Selectors are only 4 bytes long so there is a theoretical probability of a collision (e.g. two functions having the same selector).

This is comparable to the "birthday attack" : https://en.wikipedia.org/wiki/Birthday_attack
The probability of a collision when you have 93 different functions is 10^−6.
Due to the structure of the Router.sol, the solidity compiler does not prevent collisions

## Proof of Concept
https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/Router.sol#L97
```JS
  function getRouterImplementation(bytes4 sig) public view returns (address) {
        if (
            sig == NotionalProxy.batchBalanceAction.selector ||
            sig == NotionalProxy.batchBalanceAndTradeAction.selector ||
     ...

```

## Tools Used

## Recommended Mitigation Steps
Double check (perhaps via a continuous integration script / github workflow), that there are no collisions of the selectors.

