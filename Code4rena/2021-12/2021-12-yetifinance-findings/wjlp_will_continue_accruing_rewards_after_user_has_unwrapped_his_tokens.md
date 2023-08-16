## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [WJLP will continue accruing rewards after user has unwrapped his tokens](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/136) 

# Handle

kenzo


# Vulnerability details

WJLP doesn't update the inner accounting (for JOE rewards) when unwrapping user's tokens.
The user will continue to receive rewards, on the expanse of users who haven't claimed their rewards yet.

## Impact
Loss of yield for users.

## Proof of Concept
The unwrap function just withdraws JLP from MasterChefJoe, burns the user's WJLP, and sends the JLP back to the user. It does not update the inner accounting (`userInfo`). [(Code ref)](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L148:#L167)
```
    function unwrapFor(address _to, uint _amount) external override {
        _requireCallerIsAPorSP();
        _MasterChefJoe.withdraw(_poolPid, _amount);
        // msg.sender is either Active Pool or Stability Pool
        // each one has the ability to unwrap and burn WAssets they own and
        // send them to someone else
        _burn(msg.sender, _amount);
        JLP.transfer(_to, _amount);
    }
```

## Recommended Mitigation Steps
Need to keep userInfo updated. Have to take into consideration the fact that user can choose to set the reward claiming address to be a different account than the one that holds the WJLP.

