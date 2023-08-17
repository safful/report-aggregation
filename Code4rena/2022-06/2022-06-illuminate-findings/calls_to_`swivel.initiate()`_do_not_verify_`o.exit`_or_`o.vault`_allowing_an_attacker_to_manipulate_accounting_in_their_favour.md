## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Calls To `Swivel.initiate()` Do Not Verify `o.exit` or `o.vault` Allowing An Attacker To Manipulate Accounting In Their Favour](https://github.com/code-423n4/2022-06-illuminate-findings/issues/93) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L299


# Vulnerability details

## Impact

Swivel `lend()` does not validate the `o.exit` and `o.vault` for each order before making the external call to Swivel. These values determine which internal functions is [called in Swivel](https://github.com/Swivel-Finance/swivel/blob/2471ea5cda53568df5e5515153c6962f151bf358/contracts/v2/swivel/Swivel.sol#L64-L77).

The intended code path is `initiateZcTokenFillingVaultInitiate()` which takes the underlying tokens and mints zcTokens to the `Lender`. If one of the other functions is called the accounting in `lend()`. Swivel may transfer more tokens from `Lender` to `Swivel` than paid for by the caller of `lend()`.

The impact is that underlying tokens may be stolen from `Lender`.

## Proof of Concept

Consider the example where [initiateZcTokenFillingZcTokenExit()](https://github.com/Swivel-Finance/swivel/blob/2471ea5cda53568df5e5515153c6962f151bf358/contracts/v2/swivel/Swivel.sol#L162) is called. This will transfer `a - premiumFilled + fee` from `Lender` to `Swivel` rather than the expected `a + fee`.

## Recommended Mitigation Steps

In `lend()` restrict the values of `o.exit` and `o.vault` so only one case can be triggered in `Swivel.initiate()`.

