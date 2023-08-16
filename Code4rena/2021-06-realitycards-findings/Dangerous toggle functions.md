## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Dangerous toggle functions](https://github.com/code-423n4/2021-06-realitycards-findings/issues/157) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details

Usually one tries to avoid toggle functions in blockchains, because it could be that you think that the first transaction you sent was not correctly submitted (but it's just pending for a long time), or you might even be unaware that it was already sent if multiple roles can set it (like with `changeMarketApproval` / `onlyGovernors`) or if it's an msig.
This results in potentially double-toggling the state, i.e, it is set to the initial value again.

Some example functions: `changeMarketCreationGovernorsOnly`, `changeMarketApproval`, and the ones that follow.

## Impact

The outcome of toggle functions is hard to predict on blockchains due to the very async nature and lack of information about pending transactions.

## Recommended Mitigation Steps

Use functions that accept a specific value as a parameter instead.

