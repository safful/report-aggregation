## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [LPTokenMaster: CEI Pattern](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/45) 

# Handle

greiart


# Vulnerability details

## Impact

It would be better to perform the allowance check before handling the token transfer. This is in line with the best practice of the CEI (checks-effects-interactions) pattern to avoid possible re-entrancy attacks.

[https://dev.to/mxmaster2s/quick-guide-to-the-checks-effect-interactions-pattern-to-remember-when-writing-your-smart-contracts-gfk](https://dev.to/mxmaster2s/quick-guide-to-the-checks-effect-interactions-pattern-to-remember-when-writing-your-smart-contracts-gfk)

## Referenced Codelines

[https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LPTokenMaster.sol#L42-L46](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LPTokenMaster.sol#L42-L46)

## Recommended Mitigation Steps

```jsx
function transferFrom(address _sender, address _recipient, uint _amount) external returns (bool) {
    _approve(_sender, msg.sender, allowance[_sender][msg.sender] - _amount);
    _transfer(_sender, _recipient, _amount);
    return true;
}
```

