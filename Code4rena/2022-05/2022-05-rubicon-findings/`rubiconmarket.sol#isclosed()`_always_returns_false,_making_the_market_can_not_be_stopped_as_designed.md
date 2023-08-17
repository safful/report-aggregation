## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`RubiconMarket.sol#isClosed()` always returns false, making the market can not be stopped as designed](https://github.com/code-423n4/2022-05-rubicon-findings/issues/339) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconMarket.sol#L471-L473


# Vulnerability details

```solidity
    function isClosed() public pure returns (bool closed) {
        return false;
    }
```

> After close, no new buys are allowed.

Based on context and comments, when the market is closed, offers can only be cancelled (offer and buy will throw). 

However, in the current implementation, `isClosed()` always returns `false`, so the checks on whether the market is closed will always pass. (E.g: `can_offer()`, `can_buy()`, etc)

And there is a storage variable called `stopped`, but it's never been used, which seems should be used for `isClosed`.

### Recommendation

Change to:

```solidity
    function isClosed() public pure returns (bool closed) {
        return stopped;
    }
```

