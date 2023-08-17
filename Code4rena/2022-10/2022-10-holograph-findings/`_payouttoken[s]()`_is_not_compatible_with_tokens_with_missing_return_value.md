## Tags

- bug
- 2 (Med Risk)
- primary issue
- sponsor confirmed
- selected for report
- responded

# [`_payoutToken[s]()` is not compatible with tokens with missing return value](https://github.com/code-423n4/2022-10-holograph-findings/issues/456) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/src/enforcer/PA1D.sol#L317
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/src/enforcer/PA1D.sol#L340


# Vulnerability details



## Impact
Payout is blocked and tokens are stuck in contract.

## Proof of Concept
`PA1D._payoutToken()` and `PA1D._payoutTokens()` call `ERC20.transfer()` in a require-statement to send tokens to a list of payout recipients.
Some tokens do not return a bool (e.g. USDT, BNB, OMG) on ERC20 methods. But since the require-statement expects a `bool`, for such a token a `void` return will also cause a revert, despite an otherwise successful transfer. That is, the token payout will always revert for such tokens.

## Tools Used
Code inspection

## Recommended Mitigation Steps
Use [OpenZeppelin's SafeERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol), which handles the return value check as well as non-standard-compliant tokens.