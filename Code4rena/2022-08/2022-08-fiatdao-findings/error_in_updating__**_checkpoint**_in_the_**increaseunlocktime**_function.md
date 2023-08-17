## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [ERROR IN UPDATING  **_checkpoint** IN THE **increaseUnlockTime** FUNCTION](https://github.com/code-423n4/2022-08-fiatdao-findings/issues/217) 

# Lines of code

https://github.com/code-423n4/2022-08-fiatdao/blob/main/contracts/VotingEscrow.sol#L513-L514


# Vulnerability details

## Impact

The potentiel impact of this error are :

* Give wrong voting power to a user at a given block.
* Give wrong total voting power at a given block.
* Give wrong total voting power.

## Proof of Concept

The error occured in this line :
https://github.com/code-423n4/2022-08-fiatdao/blob/main/contracts/VotingEscrow.sol#L513

In the **increaseUnlockTime** function the oldLocked.end passed to the function **_checkpoint** is wrong as it is the same as the new newLock end time (called unlock_time) instead of being equal to **oldUnlockTime** .

In the given CheckpointMath.md file it is stated that checkpoint details for  **increaseUnlockTime** function should be :

|  Lock | amount | end | 
| ------------- |:-------------:|:-------------:|
| old   | owner.delegated  | owner.end  |  
| new  | owner.delegated  | T |  

BUT with this error  you get a different checkpoint details : 

|  Lock | amount | end | 
| ------------- |:-------------:|:-------------:|
| old   | owner.delegated  | T  |  
| new  | owner.delegated  | T |  

The error is illustrated in the code below :

```
        LockedBalance memory locked_ = locked[msg.sender];
        uint256 unlock_time = _floorToWeek(_unlockTime); // Locktime is rounded down to weeks
        /* @audit comment
                 the unlock_time represent the newLock end time
        */
        // Validate inputs
        require(locked_.amount > 0, "No lock");
        require(unlock_time > locked_.end, "Only increase lock end");
        require(unlock_time <= block.timestamp + MAXTIME, "Exceeds maxtime");
        // Update lock
        uint256 oldUnlockTime = locked_.end;
        locked_.end = unlock_time;
        /* @audit comment
                 The locked_ end time is update from  oldUnlockTime  ==>  unlock_time
        */
        locked[msg.sender] = locked_;
        if (locked_.delegatee == msg.sender) {
            // Undelegated lock
            require(oldUnlockTime > block.timestamp, "Lock expired");
            LockedBalance memory oldLocked = _copyLock(locked_);
            oldLocked.end = unlock_time;
            /* @audit comment
                 The oldLocked.end is set to unlock_time instead of   oldUnlockTime 
            */
            _checkpoint(msg.sender, oldLocked, locked_);
        }
```

The impact of this is when calculating the **userOldPoint.bias** in the **_checkpoint** function you get an incorrect value equal to **userNewPoint.bias** (because oldLocked.end == _newLocked.end which is wrong).

```
240        userOldPoint.bias =
241                    userOldPoint.slope *
242                    int128(int256(_oldLocked.end - block.timestamp));
```

The wrong **userOldPoint.bias** value is later used to calculate and update the bias value for the new point in **PointHistory**. 

```
359       lastPoint.bias =
360                  lastPoint.bias +
361                  userNewPoint.bias -
362                  userOldPoint.bias;

372       pointHistory[epoch] = lastPoint;
```

And added to that the wrong **oldLocked.end** is used to get oldSlopeDelta value which is used to update the **slopeChanges**.

```
271       oldSlopeDelta = slopeChanges[_oldLocked.end];

380       oldSlopeDelta = oldSlopeDelta + userOldPoint.slope;
381       if (_newLocked.end == _oldLocked.end) {
382                  oldSlopeDelta = oldSlopeDelta - userNewPoint.slope; // It was a new deposit, not extension
383        }
384       slopeChanges[_oldLocked.end] = oldSlopeDelta;
```


As the **PointHistory** and the **slopeChanges** values are used inside the functions **balanceOfAt()** ,  **_supplyAt()**,  **totalSupply()**,  **totalSupplyAt()** to calculate the voting power, THIS ERROR COULD GIVE WRONG VOTING POWER AT A GIVEN BLOCK OF A USER OR CAN GIVE WRONG TOTAL VOTING POWER.

## Tools Used

Manual Audit

## Recommended Mitigation Steps

The line 513 in the VotingEscrow.sol contract :

```
      513      oldLocked.end = unlock_time;
```

Need to be replaced with the following :

```
      513      oldLocked.end = oldUnlockTime;
```