## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [maxAmount and balance](https://github.com/code-423n4/2021-11-malt-findings/issues/357) 

# Handle

pauliax


# Vulnerability details

## Impact
I think this if check is incorrect, because in theory maxAmount parameter can be greater than totalMaltBalance:
```solidity
    if (rewards <= deployedCapital && maxAmount != totalMaltBalance) {
      // If all malt is spent we want to reset deployed capital
      deployedCapital = deployedCapital - rewards;
    } else {
      deployedCapital = 0;
    }
```

## Recommended Mitigation Steps
If my assumption is correct, the check should use balance, not maxAmount:
``solidity
  balance != totalMaltBalance
```
Another possible solution:
``solidity
  maxAmount <= totalMaltBalance
```
However, I think the best approach would be to eliminate 'balance' altogether:
```solidity
  uint256 totalMaltBalance = malt.balanceOf(address(this));

  if (totalMaltBalance == 0) {
    return 0;
  }

  (uint256 basis,) = costBasis();

  if (maxAmount > totalMaltBalance) {
    maxAmount = totalMaltBalance;
  }

  malt.safeTransfer(address(dexHandler), maxAmount);
  uint256 rewards = dexHandler.sellMalt();

  if (rewards <= deployedCapital && maxAmount < totalMaltBalance) {
    // If all malt is spent we want to reset deployed capital
    deployedCapital = deployedCapital - rewards;
  } else {
    deployedCapital = 0;
  }  
```

