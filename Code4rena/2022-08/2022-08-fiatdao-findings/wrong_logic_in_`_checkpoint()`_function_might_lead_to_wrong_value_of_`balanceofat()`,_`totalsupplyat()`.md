## Tags

- bug
- duplicate
- disagree with severity
- downgraded by judge
- QA (Quality Assurance)
- sponsor confirmed
- old-submission-method

# [Wrong logic in `_checkpoint()` function might lead to wrong value of `balanceOfAt()`, `totalSupplyAt()`](https://github.com/code-423n4/2022-08-fiatdao-findings/issues/294) 

# Lines of code

https://github.com/code-423n4/2022-08-fiatdao/blob/fece3bdb79ccacb501099c24b60312cd0b2e4bb2/contracts/VotingEscrow.sol#L257-L264
https://github.com/code-423n4/2022-08-fiatdao/blob/fece3bdb79ccacb501099c24b60312cd0b2e4bb2/contracts/VotingEscrow.sol#L372


# Vulnerability details

## Impact

In function `_checkpoint()`, new values of `userPointHistory` and `pointHistory` are override old values instead of appending to the end of the list, i.e creating new element.

The result is if we try to get `balanceOf` or `totalSupply` at current block number, it just return wrong value because values of `globalEpoch` is overrided.

## Proof of Concept

[Line 257-264](https://github.com/code-423n4/2022-08-fiatdao/blob/fece3bdb79ccacb501099c24b60312cd0b2e4bb2/contracts/VotingEscrow.sol#L257-L264)
```solidity
if (uEpoch == 0) {
    userPointHistory[_addr][uEpoch + 1] = userOldPoint;
}

userPointEpoch[_addr] = uEpoch + 1;
userNewPoint.ts = block.timestamp;
userNewPoint.blk = block.number;
userPointHistory[_addr][uEpoch + 1] = userNewPoint;
```

When `uEpoch == 0`, values of `userPointHistory` with `index = uEpoch + 1` is updated to `userOldPoint` but in line 264, values of `userPointHistory` with `index = uEpoch + 1` is overrided to `userNewPoint` which basically makes line 257-259 has no meaning.

Similarly issue with value of `pointHistory[epoch]` in [line 372](https://github.com/code-423n4/2022-08-fiatdao/blob/fece3bdb79ccacb501099c24b60312cd0b2e4bb2/contracts/VotingEscrow.sol#L372)

## Tools Used

Manual Review

## Recommended Mitigation Steps

Update logic of `_checkpoint()`, for example, use `++` operator to make sure `epoch` is increased each time it appends new element.


