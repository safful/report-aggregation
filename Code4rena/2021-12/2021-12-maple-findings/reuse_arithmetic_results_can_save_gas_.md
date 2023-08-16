## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Reuse arithmetic results can save gas ](https://github.com/code-423n4/2021-12-maple-findings/issues/66) 

# Handle

WatchPug


# Vulnerability details

https://github.com/maple-labs/debt-locker/blob/81f55907db7b23d27e839b9f9f73282184ed4744/contracts/DebtLocker.sol#L205-L215

```solidity
function _handleClaimOfRepossessed() internal returns (uint256[7] memory details_) {
    ...
    details_[0] = recoveredFunds + fundsCaptured;
    details_[1] = recoveredFunds > principalToCover ? recoveredFunds - principalToCover : 0;
    details_[2] = fundsCaptured;
    details_[5] = recoveredFunds > principalToCover ? principalToCover : recoveredFunds;
    details_[6] = principalToCover > recoveredFunds ? principalToCover - recoveredFunds : 0;

    _fundsToCapture = uint256(0);
    _repossessed    = false;

    require(ERC20Helper.transfer(fundsAsset, _pool, recoveredFunds + fundsCaptured), "DL:HCOR:TRANSFER");
}
```

`recoveredFunds + fundsCaptured` at L215 is calculated before at L205, since it's a checked arithmetic operation with two memory variables, resue the result instead of doing the arithmetic operation again can save gas.

### Recommendation

Change to:

`require(ERC20Helper.transfer(fundsAsset, _pool, details_[0]), "DL:HCOR:TRANSFER");`

