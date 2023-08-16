## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity
- resolved

# [payout doesn't fix isForeclosed state](https://github.com/code-423n4/2021-06-realitycards-findings/issues/28) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function payout of RCTreasury.sol doesn't undo the isForeclosed state of a user.
This would be possible because with a payout a user will receive funds so he can lose his isForeclosed status.

For example the function refundUser does check and update the isForeclosed status.

## Proof of Concept
// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L429
 function payout(address _user, uint256 _amount)  external override balancedBooks onlyMarkets returns (bool) {
        require(!globalPause, "Payouts are disabled");
        assert(marketPot[msgSender()] >= _amount);
        user[_user].deposit += SafeCast.toUint128(_amount);
        marketPot[msgSender()] -= _amount;
        totalMarketPots -= _amount;
        totalDeposits += _amount;
        emit LogAdjustDeposit(_user, _amount, true);
        return true;
    }

// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L447 
function refundUser(address _user, uint256 _refund)  external override onlyMarkets {
     ...
        if ( isForeclosed[_user] && user[_user].deposit > user[_user].bidRate / minRentalDayDivisor )  {
            isForeclosed[_user] = false;
            emit LogUserForeclosed(_user, false);
        }

## Tools Used

## Recommended Mitigation Steps
Check and update the isForeclosed state in the payout function

