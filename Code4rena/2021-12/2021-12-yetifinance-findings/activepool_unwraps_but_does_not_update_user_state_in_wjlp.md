## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [ActivePool unwraps but does not update user state in WJLP](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/209) 

# Handle

cmichel


# Vulnerability details

Calling `WJLP.unwrap` burns WJLP, withdraws the amount from the master chef and returns the same amount of JLP back to the `to` address.
However, it does not update the internal accounting in `WJLP` with a `_userUpdate` call.

This needs to be done on the caller side according to the comment in the `WJLP.unwrap` function:
> "Prior to this being called, the user whose assets we are burning should have their rewards updated"

This happens when being called from the `StabilityPool` but not when being called from the `ActivePool.sendCollateralsUnwrap`:

```solidity
function sendCollateralsUnwrap(address _to, address[] memory _tokens, uint[] memory _amounts, bool _collectRewards) external override returns (bool) {
    _requireCallerIsBOorTroveMorTMLorSP();
    require(_tokens.length == _amounts.length);
    for (uint i = 0; i < _tokens.length; i++) {
        if (whitelist.isWrapped(_tokens[i])) {
            // @audit this burns the tokens for _to but does not reduce their amount. so there are no tokens in WJLP masterchef but can keep claiming
            IWAsset(_tokens[i]).unwrapFor(_to, _amounts[i]);
            if (_collectRewards) {
                IWAsset(_tokens[i]).claimRewardFor(_to);
            }
        } else {
            _sendCollateral(_to, _tokens[i], _amounts[i]); // reverts if send fails
        }
    }
    return true;
}
```

## Impact
The `unwrapFor` call withdraws the tokens from the Masterchef and pays out the user, but their user balance is never decreased by the withdrawn amount.
They can still use their previous balance to claim rewards through `WJLP.claimReward` which updated their unclaimed joe reward according to the old balance.
Funds from the WJLP pool can be stolen.

## Recommended Mitigation Steps
As the comment says, make sure the user is updated before each `unwrap` call.
It might be easier and safer to have a second authorized `unwrapFor` function that accepts a `rewardOwner` parameter, the user that needs to be updated.


