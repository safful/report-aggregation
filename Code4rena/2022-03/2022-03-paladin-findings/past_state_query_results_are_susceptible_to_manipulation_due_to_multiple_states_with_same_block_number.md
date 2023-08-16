## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Past state query results are susceptible to manipulation due to multiple states with same block number](https://github.com/code-423n4/2022-03-paladin-findings/issues/20) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L466
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L492
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L644
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L663
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L917
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L961
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L993
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1148
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1164
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1184
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1199
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1225
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1250
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1260
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1287
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1293
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1324
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1352
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1357


# Vulnerability details

## Impact

4 kinds of states (`UserLock`, `TotalLock`, `Checkpoint`, `DelegateCheckpoint`) are maintained in the protocol to keep record of history. For functions that query history states, target block number is used as an index to search for the corresponding state.

However, 3 (`DelegateCheckpoint`, `TotalLock`, `UserLocks`) out of the 4 states are allowed to have multiple entries with same `fromBlock`, resulting in a one-to-many mapping between block number and history entry. This makes queried results at best imprecise, and at worst manipulatable by malicious users to present an incorrect history.

## Proof of Concept

Functions that query history states including `_getPastLock`, `getPastTotalLock`, `_getPastDelegate` perform a binary search through the array of history states to find entry matching queried block number. However, the searched arrays can contain multiple entries with the same `fromBlock`.

For example the `_lock` function pushes a new `UserLock` to `userLocks[user]` regardless of previous lock block number.

```
    function _lock(address user, uint256 amount, uint256 duration, LockAction action) internal {
        require(user != address(0)); //Never supposed to happen, but security check
        require(amount != 0, "hPAL: Null amount");
        uint256 userBalance = balanceOf(user);
        require(amount <= userBalance, "hPAL: Amount over balance");
        require(duration >= MIN_LOCK_DURATION, "hPAL: Lock duration under min");
        require(duration <= MAX_LOCK_DURATION, "hPAL: Lock duration over max");

        if(userLocks[user].length == 0){
            ...
        }
        else {
            // Get the current user Lock
            uint256 currentUserLockIndex = userLocks[user].length - 1;
            UserLock storage currentUserLock = userLocks[user][currentUserLockIndex];
            // Calculate the end of the user current lock
            uint256 userCurrentLockEnd = currentUserLock.startTimestamp + currentUserLock.duration;

            uint256 startTimestamp = block.timestamp;

            if(currentUserLock.amount == 0 || userCurrentLockEnd < block.timestamp) {
                // User locked, and then unlocked
                // or user lock expired

                userLocks[user].push(UserLock(
                    safe128(amount),
                    safe48(startTimestamp),
                    safe48(duration),
                    safe32(block.number)
                ));
            }
            else {
                // Update of the current Lock : increase amount or increase duration
                // or renew with the same parameters, but starting at the current timestamp
                require(amount >=  currentUserLock.amount,"hPAL: smaller amount");
                require(duration >=  currentUserLock.duration,"hPAL: smaller duration");

                // If the method is called with INCREASE_AMOUNT, then we don't change the startTimestamp of the Lock

                userLocks[user].push(UserLock(
                    safe128(amount),
                    action == LockAction.INCREASE_AMOUNT ? currentUserLock.startTimestamp : safe48(startTimestamp),
                    safe48(duration),
                    safe32(block.number)
                ));
                ...
            }
        ...
    }
```

This makes the history searches imprecise at best. Additionally, if a user intends to shadow his past states from queries through public search functions, it is possible to control the number of entries precisely such that binsearch returns the entry he wants to show.


## Tools Used

vim, ganache-cli

## Recommended Mitigation Steps

Adopt the same strategy as checkpoint, and modify last entry in array instead of pushing new one if it `fromBlock == block.number`


