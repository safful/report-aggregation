## Tags

- bug
- duplicate
- invalid
- 2 (Med Risk)
- sponsor confirmed

# [Possible reentrancy in balanceOf, decimals, mint](https://github.com/code-423n4/2021-08-notional-findings/issues/14) 

# Handle

tensors


# Vulnerability details

## Impact
Registering tokens that aren't properly vetted can lead to a loss of funds if the token has callbacks.

CREAM finance got hacked in a similar way because the ampleforth token had a callback
in the transfer method that wasn't noticed when they vetted it. 

## Proof of Concept
For example, the redeem function is vulnerable to such a reentrancy in the balanceOf method, since address balances aren't updated before the token call.

https://github.com/code-423n4/2021-08-notional/blob/4b51b0de2b448e4d36809781c097c7bc373312e9/contracts/internal/balances/TokenHandler.sol#L131


## Recommended Mitigation Steps
Either:
- Add nonreentrant modifiers
- Update all storage variables before making outside calls with the token
- Put steps in place for devs to properly vet tokens.

