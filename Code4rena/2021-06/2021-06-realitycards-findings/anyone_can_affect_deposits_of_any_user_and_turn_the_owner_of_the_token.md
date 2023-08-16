## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved

# [Anyone can affect deposits of any user and turn the owner of the token](https://github.com/code-423n4/2021-06-realitycards-findings/issues/119) 

# Handle

a_delamo


# Vulnerability details

## Impact

On `RCTreasury`, we have the method `collectRentUser`. This method is public, so anyone can call it using whatever user and whatever timestamp. 
So, calling this method using `user = XXXXX` and `_timeToCollectTo = type(uint256).max)`, would make `isForeclosed[user] = true`. 
```
    function collectRentUser(address _user, uint256 _timeToCollectTo)
        public
        override
        returns (
            uint256 newTimeLastCollectedOnForeclosure
        )
    {
        require(!globalPause, "Global pause is enabled");
        assert(_timeToCollectTo != 0);
        if (user[_user].lastRentCalc < _timeToCollectTo) {
            uint256 rentOwedByUser = rentOwedUser(_user, _timeToCollectTo);

            if (rentOwedByUser > 0 && rentOwedByUser > user[_user].deposit) {
                // The User has run out of deposit already.
                uint256 previousCollectionTime = user[_user].lastRentCalc;

                /*
            timeTheirDepsitLasted = timeSinceLastUpdate * (usersDeposit/rentOwed)
                                  = (now - previousCollectionTime) * (usersDeposit/rentOwed)
            */
                uint256 timeUsersDepositLasts =
                    ((_timeToCollectTo - previousCollectionTime) *
                        uint256(user[_user].deposit)) / rentOwedByUser;
                /*
            Users last collection time = previousCollectionTime + timeTheirDepsitLasted
            */
                rentOwedByUser = uint256(user[_user].deposit);
                newTimeLastCollectedOnForeclosure =
                    previousCollectionTime +
                    timeUsersDepositLasts;
                _increaseMarketBalance(rentOwedByUser, _user);
                user[_user].lastRentCalc = SafeCast.toUint64(
                    newTimeLastCollectedOnForeclosure
                );
                assert(user[_user].deposit == 0);
                isForeclosed[_user] = true;
                emit LogUserForeclosed(_user, true);
            } else {
                // User has enough deposit to pay rent.
                _increaseMarketBalance(rentOwedByUser, _user);
                user[_user].lastRentCalc = SafeCast.toUint64(_timeToCollectTo);
            }
            emit LogAdjustDeposit(_user, rentOwedByUser, false);
        }
    }
```

Now, we can do the same for all the users bidding for a specific token. 
Finally, I can become the owner of the token by just calling `newRental` and using a small price. `newRental` will iterate over all the previous bid and will remove them because there are foreclosed.


## Tools Used
Editor

## Recommended Mitigation Steps

`collectRentUser` should be private and create a new public method with `onlyOrderbook` modifier

