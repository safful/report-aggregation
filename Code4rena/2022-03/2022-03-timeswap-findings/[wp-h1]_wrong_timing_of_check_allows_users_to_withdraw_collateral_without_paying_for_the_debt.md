## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [[WP-H1] Wrong timing of check allows users to withdraw collateral without paying for the debt](https://github.com/code-423n4/2022-03-timeswap-findings/issues/16) 

# Lines of code

https://github.com/code-423n4/2022-03-timeswap/blob/00317d9a8319715a8e28361901ab14fe50d06172/Timeswap/Core/contracts/TimeswapPair.sol#L459-L490


# Vulnerability details

https://github.com/code-423n4/2022-03-timeswap/blob/00317d9a8319715a8e28361901ab14fe50d06172/Timeswap/Core/contracts/TimeswapPair.sol#L459-L490

```solidity
function pay(PayParam calldata param)
    external 
    override 
    lock 
    returns (
        uint128 assetIn, 
        uint128 collateralOut
    ) 
{
    require(block.timestamp < param.maturity, 'E202');
    require(param.owner != address(0), 'E201');
    require(param.to != address(0), 'E201');
    require(param.to != address(this), 'E204');
    require(param.ids.length == param.assetsIn.length, 'E205');
    require(param.ids.length == param.collateralsOut.length, 'E205');

    Pool storage pool = pools[param.maturity];

    Due[] storage dues = pool.dues[param.owner];
    require(dues.length >= param.ids.length, 'E205');

    for (uint256 i; i < param.ids.length;) {
        Due storage due = dues[param.ids[i]];
        require(due.startBlock != BlockNumber.get(), 'E207');
        if (param.owner != msg.sender) require(param.collateralsOut[i] == 0, 'E213');
        require(uint256(assetIn) * due.collateral >= uint256(collateralOut) * due.debt, 'E303');
        due.debt -= param.assetsIn[i];
        due.collateral -= param.collateralsOut[i];
        assetIn += param.assetsIn[i];
        collateralOut += param.collateralsOut[i];
        unchecked { ++i; }
    }
    ...
```

At L484, if there is only one `id`, and for the first and only time of the for loop, `assetIn` and `collateralOut` will be `0`, therefore `require(uint256(assetIn) * due.collateral >= uint256(collateralOut) * due.debt, 'E303');` will pass.

A attacker can call `pay()` with `param.assetsIn[0] == 0` and `param.collateralsOut[i] == due.collateral`.

### PoC

The attacker can: 

1. `borrow()` `10,000 USDC` with `1 BTC` as `collateral`;
2. `pay()` with `0 USDC` as `assetsIn` and `1 BTC` as `collateralsOut`.

As a result, the attacker effectively stole `10,000 USDC`.

### Recommendation

Change to:

```solidity
for (uint256 i; i < param.ids.length;) {
    Due storage due = dues[param.ids[i]];
    require(due.startBlock != BlockNumber.get(), 'E207');
    if (param.owner != msg.sender) require(param.collateralsOut[i] == 0, 'E213');
    due.debt -= param.assetsIn[i];
    due.collateral -= param.collateralsOut[i];
    assetIn += param.assetsIn[i];
    collateralOut += param.collateralsOut[i];
    unchecked { ++i; }
}

require(uint256(assetIn) * due.collateral >= uint256(collateralOut) * due.debt, 'E303');
...
```

