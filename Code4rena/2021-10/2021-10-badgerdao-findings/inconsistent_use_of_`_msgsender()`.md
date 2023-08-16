## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Inconsistent use of `_msgSender()`](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/54) 

# Handle

WatchPug


# Vulnerability details

`msg.sender` vs internal call of `_msgSender()`.

https://github.com/code-423n4/2021-10-badgerdao/blob/9c0ea7b3b02675211446f6c81750c5f3c0a86370/contracts/WrappedIbbtc.sol#L23-L36

```solidity
modifier onlyPendingGovernance() {
    require(msg.sender == pendingGovernance, "onlyPendingGovernance");
    _;
}

modifier onlyGovernance() {
    require(msg.sender == governance, "onlyGovernance");
    _;
}

modifier onlyOracle() {
    require(msg.sender == address(oracle), "onlyOracle");
    _;
}
```

https://github.com/code-423n4/2021-10-badgerdao/blob/9c0ea7b3b02675211446f6c81750c5f3c0a86370/contracts/WrappedIbbtcEth.sol#L123-L131

```solidity
function transfer(address recipient, uint256 amount) public virtual override returns (bool) {
    /// The _balances mapping represents the underlying ibBTC shares ("non-rebased balances")
    /// Some naming confusion emerges due to maintaining original ERC20 var names

    uint256 amountInShares = balanceToShares(amount);

    _transfer(_msgSender(), recipient, amountInShares);
    return true;
}
```

