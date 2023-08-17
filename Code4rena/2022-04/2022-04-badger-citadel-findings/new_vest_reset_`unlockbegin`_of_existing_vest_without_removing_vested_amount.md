## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [New vest reset `unlockBegin` of existing vest without removing vested amount](https://github.com/code-423n4/2022-04-badger-citadel-findings/issues/158) 

# Lines of code

https://github.com/code-423n4/2022-04-badger-citadel/blob/18f8c392b6fc303fe95602eba6303725023e53da/src/StakedCitadelVester.sol#L143
https://github.com/code-423n4/2022-04-badger-citadel/blob/18f8c392b6fc303fe95602eba6303725023e53da/src/StakedCitadelVester.sol#L109


# Vulnerability details

## Impact
When `vest` is called by xCTDL vault, the previous amount will re-lock according to the new vesting timeline. While this is as described in L127, `claimableBalance` might revert due to underflow if `vesting[recipient].claimedAmounts` > 0 because the user will need to vest the `claimedAmounts` again which should not be an expected behavior as it is already vested.

## Proof of Concept
https://github.com/code-423n4/2022-04-badger-citadel/blob/18f8c392b6fc303fe95602eba6303725023e53da/src/StakedCitadelVester.sol#L143
```
        vesting[recipient].lockedAmounts =
            vesting[recipient].lockedAmounts +
            _amount;
        vesting[recipient].unlockBegin = _unlockBegin;
        vesting[recipient].unlockEnd = _unlockBegin + vestingDuration;
```
https://github.com/code-423n4/2022-04-badger-citadel/blob/18f8c392b6fc303fe95602eba6303725023e53da/src/StakedCitadelVester.sol#L109
```
        uint256 locked = vesting[recipient].lockedAmounts;
        uint256 claimed = vesting[recipient].claimedAmounts;
        if (block.timestamp >= vesting[recipient].unlockEnd) {
            return locked - claimed;
        }
        return
            ((locked * (block.timestamp - vesting[recipient].unlockBegin)) /
                (vesting[recipient].unlockEnd -
                    vesting[recipient].unlockBegin)) - claimed;
```

## Recommended Mitigation Steps
Reset claimedAmounts on new vest
```
        vesting[recipient].lockedAmounts =
            vesting[recipient].lockedAmounts - 
            vesting[recipient].claimedAmounts +
            _amount;
        vesting[recipient].claimedAmounts = 0
        vesting[recipient].unlockBegin = _unlockBegin;
        vesting[recipient].unlockEnd = _unlockBegin + vestingDuration;
```

