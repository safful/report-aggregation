## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [wrong condition checking in price calculation](https://github.com/code-423n4/2022-03-biconomy-findings/issues/105) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityProviders.sol#L180-L186


# Vulnerability details

## Impact
The `getTokenPriceInLPShares` function calculates the token price in LP shares, but it checks a wrong condition - if supposed to return `BASE_DIVISOR` if the total reserve is zero, not if the total shares minted is zero. This might leads to a case where the price is calculated incorrectly, or a division by zero is happening. 

## Proof of Concept
This is the wrong function implementation:
```sol
function getTokenPriceInLPShares(address _baseToken) public view returns (uint256) {
    uint256 supply = totalSharesMinted[_baseToken];
    if (supply > 0) {
        return totalSharesMinted[_baseToken] / totalReserve[_baseToken];
    }
    return BASE_DIVISOR;
}
```
This function is used in this contract only in the removeLiquidity and claimFee function, so it's called only if funds were already deposited and totalReserve is not zero, but it can be problematic when other contracts will use this function (it's a public view function so it might get called from outside of the contract).

## Recommended Mitigation Steps
The correct code should be:
```sol
function getTokenPriceInLPShares(address _baseToken) public view returns (uint256) {
    uint256 reserve = totalReserve[_baseToken];
    if (reserve > 0) {
        return totalSharesMinted[_baseToken] / totalReserve[_baseToken];
    }
    return BASE_DIVISOR;
}
```

