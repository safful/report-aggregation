## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [avoid using 'else' code can save gas in function pendingPremium](https://github.com/code-423n4/2022-01-insure-findings/issues/111) 

# Handle

Tomio


# Vulnerability details

## Impact
by changing the code from `if (_credit == 0) {` to `if (_credit != 0) {` and remove the else we can save gas when contract is deploy and we can save gas when `_credit` is equal to 0. because if `_credit` equal to 0 the original function will return 0 which a default value for uint256

## Proof of Concept
Before: https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L776
// gas 24307

After:
```
function pendingPremium(address _index)
        external
        view
        returns (uint256)
    {
        uint256 _credit = indicies[_index].credit;
       if (_credit != 0) {
            return
                _sub(
                    (_credit * rewardPerCredit) / MAGIC_SCALE_1E6,
                    indicies[_index].rewardDebt
                );
        }
    }
```
// gas 24286


## Tools Used
Remix

## Recommended Mitigation Steps

