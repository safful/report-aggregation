## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Missing OOB check in `changeReceiverAlloc`](https://github.com/code-423n4/2021-12-nftx-findings/issues/181) 

# Handle

gzeon


# Vulnerability details

## Impact
`changeReceiverAlloc` did not check if the idx exists unlike other functions in the same contract

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L93
```
  function changeReceiverAlloc(uint256 _receiverIdx, uint256 _allocPoint) public override virtual onlyOwner {
    FeeReceiver storage feeReceiver = feeReceivers[_receiverIdx];
    allocTotal -= feeReceiver.allocPoint;
    feeReceiver.allocPoint = _allocPoint;
    allocTotal += _allocPoint;
    emit UpdateFeeReceiverAlloc(feeReceiver.receiver, _allocPoint);
  }
```

## Recommended Mitigation Steps
```require(_receiverIdx < feeReceivers.length, "FeeDistributor: Out of bounds");```

