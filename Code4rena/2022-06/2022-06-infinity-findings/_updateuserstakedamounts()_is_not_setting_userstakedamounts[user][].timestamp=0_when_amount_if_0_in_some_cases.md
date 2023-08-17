## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [_updateUserStakedAmounts() is not setting userstakedAmounts[user][].timestamp=0 when amount if 0 in some cases](https://github.com/code-423n4/2022-06-infinity-findings/issues/116) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/staking/InfinityStaker.sol#L287-L325


# Vulnerability details

## Impact
function `_updateUserStakedAmounts()` is supposed to update user staked amounts and it sets `userstakedAmounts[user][].timestamp=0` when `userstakedAmounts[user][].amount` is `0x0`. but there are some cases that code logic don't handle them and `amount` become `0x0` and code don't set `timestamp` to `0x0`.
so any logic that is depended on `timestamp==0` when `amount==0` could fail, to my understanding setting `timestamp` is for gas efficiency.

## Proof of Concept
This is `_updateUserStakedAmounts()` code:
```
  /** @notice Update user staked amounts for different duration on unstake
    * @dev A more elegant recursive function is possible but this is more gas efficient
   */
  function _updateUserStakedAmounts(
    address user,
    uint256 amount,
    uint256 noVesting,
    uint256 vestedThreeMonths,
    uint256 vestedSixMonths,
    uint256 vestedTwelveMonths
  ) internal {
    if (amount > noVesting) {
      userstakedAmounts[user][Duration.NONE].amount = 0;
      userstakedAmounts[user][Duration.NONE].timestamp = 0;
      amount = amount - noVesting;
      if (amount > vestedThreeMonths) {
        userstakedAmounts[user][Duration.THREE_MONTHS].amount = 0;
        userstakedAmounts[user][Duration.THREE_MONTHS].timestamp = 0;
        amount = amount - vestedThreeMonths;
        if (amount > vestedSixMonths) {
          userstakedAmounts[user][Duration.SIX_MONTHS].amount = 0;
          userstakedAmounts[user][Duration.SIX_MONTHS].timestamp = 0;
          amount = amount - vestedSixMonths;
          if (amount > vestedTwelveMonths) {
            userstakedAmounts[user][Duration.TWELVE_MONTHS].amount = 0;
            userstakedAmounts[user][Duration.TWELVE_MONTHS].timestamp = 0;
          } else {
            userstakedAmounts[user][Duration.TWELVE_MONTHS].amount -= amount;
          }
        } else {
          userstakedAmounts[user][Duration.SIX_MONTHS].amount -= amount;
        }
      } else {
        userstakedAmounts[user][Duration.THREE_MONTHS].amount -= amount;
      }
    } else {
      userstakedAmounts[user][Duration.NONE].amount -= amount;
    }
  }
```
for example if `amount=noVesting` then code would execute line: `userstakedAmounts[user][Duration.NONE].amount -= amount;` which sets the `userstakedAmounts[user][Duration.NONE].amount` to `0x0` but `userstakedAmounts[user][Duration.NONE].timestamp` won't change.
As as in all other logics when `amount` is `0x0` code set `timestamp` to `0x0` too but here that logic is not happening for this cases (amount equal to `noVesting` or `noVesting + vestedThreeMonths` or ...).

## Tools Used
VIM

## Recommended Mitigation Steps
change if conditions from `>` to `>=`, so for equal case the code set `timestamp` to `0x0` too.

