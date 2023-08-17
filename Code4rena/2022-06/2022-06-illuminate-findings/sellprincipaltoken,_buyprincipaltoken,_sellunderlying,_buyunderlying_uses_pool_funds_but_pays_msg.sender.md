## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [sellPrincipalToken, buyPrincipalToken, sellUnderlying, buyUnderlying uses pool funds but pays msg.sender](https://github.com/code-423n4/2022-06-illuminate-findings/issues/21) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/3ca41a9f529980b17fdc67baf8cbee5a8035afab/marketplace/MarketPlace.sol#L136-L189


# Vulnerability details

## Impact
Fund loss from marketplace

## Proof of Concept
sellPrincipalToken, buyPrincipalToken, sellUnderlying, buyUnderlying are all unpermissioned and use marketplace funds to complete the action but send the resulting tokens to msg.sender. This means that any address can call these functions and steal the resulting funds

## Tools Used

## Recommended Mitigation Steps
All functions should use safetransfer to get funds from msg.sender not from marketplace

