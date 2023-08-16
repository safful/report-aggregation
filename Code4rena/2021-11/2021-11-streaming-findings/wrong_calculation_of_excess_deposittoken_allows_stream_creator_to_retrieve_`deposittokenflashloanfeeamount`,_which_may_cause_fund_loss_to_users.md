## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Wrong calculation of excess depositToken allows stream creator to retrieve `depositTokenFlashloanFeeAmount`, which may cause fund loss to users](https://github.com/code-423n4/2021-11-streaming-findings/issues/241) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L654-L654

```solidity=654
uint256 excess = ERC20(token).balanceOf(address(this)) - (depositTokenAmount - redeemedDepositTokens);
```

In the current implementation, `depositTokenFlashloanFeeAmount` is not excluded when calculating `excess` depositToken. Therefore, the stream creator can call `recoverTokens(depositToken, recipient)` and retrieve `depositTokenFlashloanFeeAmount` if there are any.

As a result:

- When the protocol `governance` calls `claimFees()` and claim accumulated `depositTokenFlashloanFeeAmount`, it may fail due to insufficient balance of depositToken.
- Or, part of users' funds (depositToken) will be transferred to the protocol `governance` as fees, causing some users unable to withdraw or can only withdraw part of their deposits.

### PoC

Given:

- `feeEnabled`: true
- `feePercent`: 10 (0.1%)

1. Alice deposited `1,000,000` depositToken;
2. Bob called `flashloan()` and borrowed `1,000,000` depositToken, then repaid `1,001,000`;
3. Charlie deposited `1,000` depositToken;
4. After `endDepositLock`, Alice called `claimDepositTokens()` and withdrawn `1,000,000` depositToken;
5. `streamCreator` called `recoverTokens(depositToken, recipient)` and retrieved `1,000` depositToken `(2,000 - (1,001,000 - 1,000,000))`;
6. `governance` called `claimFees()` and retrieved another `1,000` depositToken;
7. Charlie tries to `claimDepositTokens()` but since the current balanceOf depositToken is `0`, the transcation always fails, and Charlie loses all the depositToken.

### Recommendation

Change to:

```solidity=654
uint256 excess = ERC20(token).balanceOf(address(this)) - (depositTokenAmount - redeemedDepositTokens) - depositTokenFlashloanFeeAmount;
```

