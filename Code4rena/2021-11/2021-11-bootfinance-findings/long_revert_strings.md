## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Long Revert Strings](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/23) 

# Handle

ye0lde


# Vulnerability details

## Impact

Shortening revert strings to fit in 32 bytes will decrease deployment time gas and will decrease runtime gas when the revert condition has been met.  

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

## Proof of Concept

Revert strings > 32 bytes are here:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/customswap/contracts/Swap.sol#L149-L150
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/customswap/contracts/SwapUtils.sol#L1625
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/customswap/contracts/SwapUtils.sol#L1679

https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/Vesting.sol#L105
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/Vesting.sol#L194-L197

https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L152
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L162

## Tools Used
Visual Studio Code

## Recommended Mitigation Steps
Shorten the revert strings to fit in 32 bytes.



