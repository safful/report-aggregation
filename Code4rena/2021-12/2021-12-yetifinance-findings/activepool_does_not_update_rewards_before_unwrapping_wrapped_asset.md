## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [ActivePool does not update rewards before unwrapping wrapped asset](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/150) 

# Handle

kenzo


# Vulnerability details

When ActivePool sends collateral which is a wrapped asset, it first unwraps the asset, and only after that updates the rewards.
This should be done in opposite order. As a comment in WJLP's `unwrapFor` rightfully mentions - "Prior to this being called, the user whose assets we are burning should have their rewards updated".

## Impact
Lost yield for user.

## Proof of Concept
In ActivePool's `sendCollateralsUnwrap` (which is used throughout the protocol), it firsts unwraps the asset, and only afterwards calls `claimRewardFor` which will update the rewards:
[(Code ref)](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/ActivePool.sol#L186:#L188)
```
IWAsset(_tokens[i]).unwrapFor(_to, _amounts[i]);
if (_collectRewards) {
        IWAsset(_tokens[i]).claimRewardFor(_to);
}
```
`claimRewardFor` will end up calling `_userUpdate`: [(Code ref)](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L246:#L263)
```
    function _userUpdate(address _user, uint256 _amount, bool _isDeposit) private returns (uint pendingJoeSent) {
        uint256 accJoePerShare = _MasterChefJoe.poolInfo(_poolPid).accJoePerShare;
        UserInfo storage user = userInfo[_user];
        if (user.amount > 0) {
            user.unclaimedJOEReward = user.amount.mul(accJoePerShare).div(1e12).sub(user.rewardDebt);
        }
        if (_isDeposit) {
            user.amount = user.amount.add(_amount);
        } else {
            user.amount = user.amount.sub(_amount);
        }
        user.rewardDebt = user.amount.mul(accJoePerShare).div(1e12);
    }
```
Now, as ActivePool has already called `unwrapFor` and has burnt the user's tokens, and let's assume they all were used as collateral, it means user.amount=0*, and the user's unclaimedJOEReward won't get updated to reflect the rewards from the last user update.
This is why, indeed as the comment in `unwrapFor` says, user's reward should be updated prior to that.

*Note: at the moment `unwrapFor` doesn't updates the user's user.amount, but as I detailed in another issue, that's a bug, as that means the user will continue accruing rewards even after his JLP were removed from the protocol.

## Recommended Mitigation Steps
Change the order of operations in `sendCollateralsUnwrap` to first send the updated rewards and then unwrap the asset.
You can also consider adding to the beginning of `unwrapFor` a call to `_userUpdate(_to, 0, true)` to make sure the rewards are updated before unwrapping.
Note: as user can choose to have JOE rewards accrue to a different address than the address that uses WJLP as collateral, you'll have to make sure you update the current accounts. I'll detail this in another issue.

