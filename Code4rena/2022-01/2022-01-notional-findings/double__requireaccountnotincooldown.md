## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Double _requireAccountNotInCoolDown](https://github.com/code-423n4/2022-01-notional-findings/issues/214) 

# Handle

Tomio


# Vulnerability details

## Impact
The check to make sure account is not in cool down is happening twice, on _mint() and _beforeTokenTransfer(), _beforeTokenTransfer() already has a _requireAccountNotInCoolDown() an the _mint() inside erc20upgradable will call the _beforeTokenTransfer(), this can make unnecessary call in the https://github.com/code-423n4/2022-01-notional/blob/main/contracts/sNOTE.sol#L328.

## Proof of Concept
https://github.com/code-423n4/2022-01-notional/blob/main/contracts/sNOTE.sol#L328.

## Tools Used

## Recommended Mitigation Steps

