## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`UnionToken` should check whitelist on `from`?](https://github.com/code-423n4/2021-10-union-findings/issues/69) 

# Handle

cmichel


# Vulnerability details

The `UnionToken` can check for a whitelist on each transfer in `_beforeTokenTransfer`:

```solidity
if (whitelistEnabled) {
    require(isWhitelisted(msg.sender) || to == address(0), "Whitelistable: address not whitelisted");
}
```

This whitelist is checked on `msg.sender` not on `from`, the token owner.

## Impact
A single whitelisted account can act as an operator (everyone calls `unionToken.allow(operator, max)` where the operator is a whitelisted trusted smart contract) for all other accounts. This essentially bypasses the whitelist.

## Recommended Mitigation Steps
Think about if the whitelist on `msg.sender` is correct or if it should be on `from`.

