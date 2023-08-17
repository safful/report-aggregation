## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [FraxlendPair.changeFee() doesn't update interest before changing fee.](https://github.com/code-423n4/2022-08-frax-findings/issues/236) 

# Lines of code

https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPair.sol#L215-L222


# Vulnerability details

## Impact
This function is changing the protocol fee that is used during interest calculation [here](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L477-L488).

But it doesn't update interest before changing the fee so the `_feesAmount` will be calculated wrongly.


## Proof of Concept
As we can see during [pause()](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPair.sol#L326) and [unpause()](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPair.sol#L335), `_addInterest()` must be called before any changes.

But with the [changeFee()](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPair.sol#L215), it doesn't update interest and the `_feesAmount` might be calculated wrongly.

- At time `T1`, [_currentRateInfo.feeToProtocolRate = F1](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L477).
- At `T2`, the owner had changed the fee to `F2`.
- At `T3`, [_addInterest()](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L409) is called during `deposit()` or other functions.
- Then [during this calculation](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairCore.sol#L477-L488), `F1` should be applied from `T1` to `T2` and `F2` should be applied from `T2` and `T3`. But it uses `F2` from `T1` to `T2`.


## Tools Used
Manual Review


## Recommended Mitigation Steps
Recommend modifying `changeFee()` like below.

```
function changeFee(uint32 _newFee) external whenNotPaused {
    if (msg.sender != TIME_LOCK_ADDRESS) revert OnlyTimeLock();
    if (_newFee > MAX_PROTOCOL_FEE) {
        revert BadProtocolFee();
    }

    _addInterest(); //+++++++++++++++++++++++++++++++++

    currentRateInfo.feeToProtocolRate = _newFee;
    emit ChangeFee(_newFee);
}
```