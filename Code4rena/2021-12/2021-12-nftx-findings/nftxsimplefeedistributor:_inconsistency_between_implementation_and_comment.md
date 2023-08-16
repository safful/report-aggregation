## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [NFTXSimpleFeeDistributor: Inconsistency between implementation and comment](https://github.com/code-423n4/2021-12-nftx-findings/issues/222) 

# Handle

GreyArt


# Vulnerability details

## Impact

In the `_sendForReceiver()` function, there are 2 comments:
`// If the receive is not properly processed, send it to the treasury instead.` 

and

`// If the allowance has not been spent, it means we can pass it forward to next`

which are contradictory in nature, except for the case of the last receiver in the feeReceivers array.

Looking at the `distribute()` function implementation, should the `receiveRewards()` function return false, fail, or if the `transferFrom()` was not called in its implementation, the rewards will be given to the next receiver, and not the treasury.

```jsx
// Note: some irrelevant lines were omitted
for (uint256 i = 0; i < length; i++) {
  uint256 amountToSend = leftover + ((tokenBalance * _feeReceiver.allocPoint) / allocTotal);
  bool complete = _sendForReceiver(_feeReceiver, vaultId, _vault, amountToSend);
  if (!complete) {
    leftover = amountToSend;
  } else {
    leftover = 0;
	}
}
```

## Recommended Mitigation Steps

Based on the implementation, the comment `// If the receive is not properly processed, send it to the treasury instead.` should be edited or removed.

