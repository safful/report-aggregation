## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Insufficient input validation](https://github.com/code-423n4/2021-12-maple-findings/issues/45) 

# Handle

WatchPug


# Vulnerability details

https://github.com/maple-labs/debt-locker/blob/81f55907db7b23d27e839b9f9f73282184ed4744/contracts/DebtLocker.sol#L85-L89

```solidity=85
function setAllowedSlippage(uint256 allowedSlippage_) external override whenProtocolNotPaused {
        require(msg.sender == _getPoolDelegate(), "DL:SAS:NOT_PD");

        emit AllowedSlippageSet(_allowedSlippage = allowedSlippage_);
    }
```

Considering that `_allowedSlippage` is a crucial settings for `getExpectedAmount()`, it's necessary to add `require(_allowedSlippage <  10000, "...")` to validate the input.

If `_allowedSlippage` is misconfigured to a value > `10000`, `getExpectedAmount()` will always revert.

### Recommendation

Change to:

```solidity=85
function setAllowedSlippage(uint256 allowedSlippage_) external override whenProtocolNotPaused {
        require(msg.sender == _getPoolDelegate(), "DL:SAS:NOT_PD");
        require(_allowedSlippage <  10000, "!slippage")

        emit AllowedSlippageSet(_allowedSlippage = allowedSlippage_);
    }
```

