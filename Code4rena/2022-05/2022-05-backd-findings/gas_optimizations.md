## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-05-backd-findings/issues/170) 

## `> 0` is less efficient than `!= 0` for uint in require condition
Ref: https://twitter.com/GalloDaSballo/status/1485430908165443590
```
protocol/contracts/tokenomics/AmmConvexGauge.sol:158:        require(amount > 0, Error.INVALID_AMOUNT);
protocol/contracts/tokenomics/AmmConvexGauge.sol:171:        require(amount > 0, Error.INVALID_AMOUNT);
protocol/contracts/tokenomics/VestedEscrow.sol:84:        require(unallocatedSupply > 0, "No reward tokens in contract");
protocol/contracts/tokenomics/KeeperGauge.sol:140:        require(totalClaimable > 0, Error.ZERO_TRANSFER_NOT_ALLOWED);
protocol/contracts/tokenomics/AmmGauge.sol:104:        require(amount > 0, Error.INVALID_AMOUNT);
protocol/contracts/tokenomics/AmmGauge.sol:125:        require(amount > 0, Error.INVALID_AMOUNT);
```

## Float multiplication optimization
We can use the following function to save gas on float multiplications
```
// out = x * y unchecked{/} z
function fmul(uint256 x, uint256 y, uint256 z) internal pure returns(uint256 out){
assembly{
if iszero(eq(div(mul(x,y),x),y)) {revert(0,0)}
out := div(mul(x,y),z)
}
}
```

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/VestedEscrow.sol#L156-L157

```solidity
        return Math.min((locked * elapsed) / totalTime, locked);

```