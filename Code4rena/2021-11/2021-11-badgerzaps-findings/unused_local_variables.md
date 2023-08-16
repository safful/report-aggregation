## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)

# [Unused local variables](https://github.com/code-423n4/2021-11-badgerzaps-findings/issues/39) 

# Handle

WatchPug


# Vulnerability details

Unused local variables in contracts increase contract size and gas usage at deployment.

Instances include:

https://github.com/Badger-Finance/ibbtc/blob/d8b95e8d145eb196ba20033267a9ba43a17be02c/contracts/Zap.sol#L177-L179

```solidity=177
function calcMintWithRen(uint amount) public view returns(uint poolId, uint idx, uint bBTC, uint fee) {
    uint _ibbtc;
    uint _fee;
    ...
```

https://github.com/Badger-Finance/ibbtc/blob/d8b95e8d145eb196ba20033267a9ba43a17be02c/contracts/Zap.sol#L196-L198

```solidity=196
function calcMintWithWbtc(uint amount) public view returns(uint poolId, uint idx, uint bBTC, uint fee) {
    uint _ibbtc;
    uint _fee;
    ...
```

https://github.com/Badger-Finance/ibbtc/blob/d8b95e8d145eb196ba20033267a9ba43a17be02c/contracts/Zap.sol#L274-L275

```solidity=274
uint _fee;
uint _ren;
```

https://github.com/Badger-Finance/ibbtc/blob/d8b95e8d145eb196ba20033267a9ba43a17be02c/contracts/Zap.sol#L295-L296

```solidity=295
uint _fee;
uint _wbtc;
```

