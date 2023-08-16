## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Code Style: private/internal function names should be prefixed with `_`](https://github.com/code-423n4/2021-10-covalent-findings/issues/55) 

# Handle

WatchPug


# Vulnerability details

Here are some examples that the code style of function names does not follow the best practices:

- transferToContract()
https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L74

- transferToContract()
https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L82

- updateGlobalExchangeRate()
https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L118

- updateValidator()
https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L134

- sharesToTokens()
https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L157-L164

