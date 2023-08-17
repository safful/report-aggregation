## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- valid

# [Locked splits can be updated](https://github.com/code-423n4/2022-07-juicebox-findings/issues/278) 

# Lines of code

https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSplitsStore.sol#L213-L220


# Vulnerability details

## Impact

The check if the newly provided project splits contain the currently locked splits does not check the `JBSplit` struct properties `preferClaimed` and `preferAddToBalance`.

According to the docs in `JBSplit.sol`, _"...if the split should be unchangeable until the specified time, with the exception of extending the locked period."_, locked sets are unchangeable.

However, locked sets with either `preferClaimed` or `preferAddToBalance` set to true can have their bool values overwritten by supplying the same split just with different bool values.

## Proof of Concept

[JBSplitsStore.sol#L213-L220](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBSplitsStore.sol#L213-L220)

```solidity
// Check for sameness.
if (
    _splits[_j].percent == _currentSplits[_i].percent &&
    _splits[_j].beneficiary == _currentSplits[_i].beneficiary &&
    _splits[_j].allocator == _currentSplits[_i].allocator &&
    _splits[_j].projectId == _currentSplits[_i].projectId &&
    // Allow lock extention.
    _splits[_j].lockedUntil >= _currentSplits[_i].lockedUntil
) _includesLocked = true;
```

The check for sameness does not check the equality of the struct properties `preferClaimed` and `preferAddToBalance`.

## Tools Used

Manual review

## Recommended mitigation steps

Add two additional sameness checks for `preferClaimed` and `preferAddToBalance`:

```solidity
// Check for sameness.
if (
    _splits[_j].percent == _currentSplits[_i].percent &&
    _splits[_j].beneficiary == _currentSplits[_i].beneficiary &&
    _splits[_j].allocator == _currentSplits[_i].allocator &&
    _splits[_j].projectId == _currentSplits[_i].projectId &&
    _splits[_j].preferClaimed == _currentSplits[_i].preferClaimed && // @audit-info add check for sameness for property `preferClaimed`
    _splits[_j].preferAddToBalance == _currentSplits[_i].preferAddToBalance && // @audit-info add check for sameness for property `preferAddToBalance`
    // Allow lock extention.
    _splits[_j].lockedUntil >= _currentSplits[_i].lockedUntil
) _includesLocked = true;
```


