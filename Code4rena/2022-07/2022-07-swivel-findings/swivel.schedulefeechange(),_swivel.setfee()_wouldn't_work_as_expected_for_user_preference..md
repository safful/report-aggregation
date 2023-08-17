## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- resolved
- sponsor confirmed

# [Swivel.scheduleFeeChange(), Swivel.setFee() wouldn't work as expected for user preference.](https://github.com/code-423n4/2022-07-swivel-findings/issues/118) 

# Lines of code

https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Swivel/Swivel.sol#L473
https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Swivel/Swivel.sol#L495


# Vulnerability details

## Impact
Swivel.scheduleFeeChange(), Swivel.setFee() wouldn't work as expected for user preference.

Users can't react properly after [ScheduleFeeChange() event](https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Swivel/Swivel.sol#L477) because they don't know whether the new fee settings would be better/worse for them.


## Proof of Concept
According to [this explanation](https://github.com/code-423n4/2022-07-swivel#admin-privileges), these functions are to ensure users can feel comfortable.

Btw with Swivel.scheduleFeeChange(), it emits only when to change fee settings without detailed values.

So users don't know whether the new fee settings will be better or worse for them.

Even if the admin is going to set larger feenominators for lower fee percent, users don't know that until actual fees are set using setFee() and such delays are almost meaningless for users.

I think we should announce the detailed fee settings with Swivel.scheduleFeeChange() function so that users can react accordingly after checking new fee settings.


## Tools Used
Solidity Visual Developer of VSCode


## Recommended Mitigation Steps
Recommend adding an additional array - pendingFeenominators [here](https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Swivel/Swivel.sol#L37).

```
uint16[4] public pendingFeenominators;
```

And scheduleFeeChange() function should have i, d parameters same as current setFee() function so that pendingFeenominators save new settings. (Also keep original fee settings if some indexs aren't updated.)

After that, we can call setFee() without any params and feenominators will be replaced with pendingFeenominators.

