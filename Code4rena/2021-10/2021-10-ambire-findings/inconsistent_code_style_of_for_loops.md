## Tags

- bug
- sponsor confirmed
- 0 (Non-critical)
- resolved

# [Inconsistent code style of for loops](https://github.com/code-423n4/2021-10-ambire-findings/issues/45) 

# Handle

WatchPug


# Vulnerability details

Most of the for loops in the codebase use `<` to control the loop:

```solidity
for (uint i=0; i<len; i++) {
```

However, in `Zapper.sol`, all 7 for loops are using `!=`:

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L79-L79

```solidity
for (uint i=0; i!=spenders.length; i++) {
```

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L87-L87


https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L110-L110

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L126-L126

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L131-L131

Using `for (uint i=0; i!=len; i++) {}` to control for loops introduces inconsistent code style.

### Recommendation

Change from `!=` to `<` for all for loops.

