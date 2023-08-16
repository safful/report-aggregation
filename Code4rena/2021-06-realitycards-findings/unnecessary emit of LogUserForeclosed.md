## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [unnecessary emit of LogUserForeclosed](https://github.com/code-423n4/2021-06-realitycards-findings/issues/27) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function deposit of RCTreasury.sol resets the isForeclosed state and emits LogUserForeclosed, if the use have enough funds.
However this also happens if the user is not Foreclosed and so the emit is redundant and confusing.

## Proof of Concept
// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L279
function deposit(uint256 _amount, address _user) public override balancedBooks returns (bool) {
   ....
        // this deposit could cancel the users foreclosure
        if ( (user[_user].deposit + _amount) > (user[_user].bidRate / minRentalDayDivisor) ) {
            isForeclosed[_user] = false;
            emit LogUserForeclosed(_user, false);
        }
        return true;
    }

## Tools Used

## Recommended Mitigation Steps
Only do the emit when isForeclosed was true

