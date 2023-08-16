## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [`Zapper.sol#wrapETH()` Use `WETH.deposit` can save some gas](https://github.com/code-423n4/2021-10-ambire-findings/issues/27) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L137-L140

```solidity
function wrapETH() payable external {
    // TODO: it may be slightly cheaper to call deposit() directly
    payable(WETH).transfer(msg.value);
}
```

### Recommendation

Change to:

```solidity
interface IWETH {
    function deposit() external payable;
}
function wrapETH() payable external {
    IWETH(WETH).deposit{ value: msg.value }();
}
```

