## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/156) 

# Handle

WatchPug


# Vulnerability details

For the arithmetic operations that will never over/underflow, using the unchecked directive (Solidity v0.8 has default overflow/underflow checks) can save some gas from the unnecessary internal over/underflow checks.

For example:

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L149-L150

```solidity
        require(block.timestamp < maturity, 'E202');
        require(maturity - block.timestamp < 0x100000000, 'E208');
```

 `maturity - block.timestamp` will never underflow.


https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/libraries/WithdrawMath.sol#L31-L33

```solidity
        if (state.reserves.asset >= state.totalClaims.bond) return collateralOut;
        uint256 deficit = state.totalClaims.bond;
        deficit -= state.reserves.asset;
```

`deficit -= state.reserves.asset` will never underflow.


https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/MsgValue.sol#L12-L12

```solidity
if (msg.value > value) ETH.transfer(payable(msg.sender), msg.value - value);
```

`msg.value - value` will never underflow.

