## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Cache array length in for loops can save gas](https://github.com/code-423n4/2021-10-ambire-findings/issues/26) 

# Handle

WatchPug


# Vulnerability details

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

Caching the array length in the stack saves around 3 gas per iteration.

Instances include:

- `Zapper.sol#constructor()` https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L79-L79

- `Zapper.sol#approveMaxMany()` https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L87-L87

- `Zapper.sol#exchangeV2()` https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L104-L104

- `Zapper.sol#wrapLending()` https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L126-L126

- `Zapper.sol#unwrapLending()` https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L131

- `QuickAccManager.sol#sendTxns()` https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/QuickAccManager.sol#L162-L162

