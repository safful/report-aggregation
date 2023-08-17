## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- selected for report
- edited-by-warden
- responded

# [Failed job can't be recovered. NFT may be lost.](https://github.com/code-423n4/2022-10-holograph-findings/issues/102) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L329
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L419-L429


# Vulnerability details

## Impact
Failed job can't be recovered. NFT may be lost.

## Proof of Concept
```solidity
function executeJob(bytes calldata bridgeInRequestPayload) external payable {
...
delete _operatorJobs[hash];
...
    try
      HolographOperatorInterface(address(this)).nonRevertingBridgeCall{value: msg.value}(
        msg.sender,
        bridgeInRequestPayload
      )
    {
      /// @dev do nothing
    } catch {
      _failedJobs[hash] = true;
      emit FailedOperatorJob(hash);
    }
}
```

First, it will `delete _operatorJobs[hash];` to have it not replayable.

Next, assume nonRevertingBridgeCall failed. NFT won't be minted and the catch block is entered.

_failedJobs[hash] is set to true and event is emitted

Notice that _operatorJobs[hash] has been deleted, so this job is not replayable. This mean NFT is lost forever since we can't retry executeJob.

## Recommended Mitigation Steps
Move `delete _operatorJobs[hash];` to the end of function executeJob covered in `if (!_failedJobs[hash])`

```solidity
...
if (!_failedJobs[hash]) delete _operatorJobs[hash];
...
```

But this implementation is not safe. The selected operator may get slashed. Additionally, you may need to check _failedJobs flag to allow retry for only the selected operator.