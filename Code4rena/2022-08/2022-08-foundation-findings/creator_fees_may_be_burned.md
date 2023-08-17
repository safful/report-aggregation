## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Creator fees may be burned](https://github.com/code-423n4/2022-08-foundation-findings/issues/31) 

# Lines of code

https://github.com/code-423n4/2022-08-foundation/blob/7d6392498e8f3b8cdc22beb582188ffb3ed25790/contracts/mixins/shared/MarketFees.sol#L128


# Vulnerability details

## Impact
`royaltyInfo`, `getRoyalties`, or `getFeeRecipients` may return `address(0)` as the recipient address. While the value 0 is correctly handled for the royalties itself, it is not for the address. In such a case, the ETH amount will be sent to `address(0)`, i.e. it is burned and lost.

## Recommended Mitigation Steps
In your logic for determining the recipients, treat `address(0)` as if no recipient was returned such that the other priorities / methods take over.