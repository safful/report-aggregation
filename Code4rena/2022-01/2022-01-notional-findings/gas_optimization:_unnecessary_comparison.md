## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimization: Unnecessary comparison](https://github.com/code-423n4/2022-01-notional-findings/issues/161) 

# Handle

gzeon


# Vulnerability details

## Impact
In `_requireAccountNotInCoolDown`
if `block.timestamp < coolDown.redeemWindowEnd`, we must have `coolDown.redeemWindowEnd > 0` hence `0 < coolDown.redeemWindowBegin`

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L308
```
        bool isInCoolDown = (0 < coolDown.redeemWindowBegin && block.timestamp < coolDown.redeemWindowEnd);
        require(!isInCoolDown, "Account in Cool Down");
```
to 
```
        require(block.timestamp >= coolDown.redeemWindowEnd, "Account in Cool Down");
```

