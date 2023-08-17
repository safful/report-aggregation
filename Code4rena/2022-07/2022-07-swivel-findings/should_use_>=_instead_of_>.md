## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [should use >= instead of >](https://github.com/code-423n4/2022-07-swivel-findings/issues/21) 

# Lines of code

https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/VaultTracker/VaultTracker.sol#L86
https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/VaultTracker/VaultTracker.sol#L158


# Vulnerability details

## should use >= instead of > 

### description

https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/VaultTracker/VaultTracker.sol#L86
https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/VaultTracker/VaultTracker.sol#L158

the comparison should be 'a >= vlt.notional' instead of a > vlt.notional

otherwise dust amounts will always be left in vlt.notional when calling `removeNotional()` or `transferNotionalFrom()`


