## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [debts calculation is not accurate](https://github.com/code-423n4/2021-10-mochi-findings/issues/25) 

# Handle

gpersoon


# Vulnerability details

## Impact
The value of the global variable debts in the contract MochiVault.sol is calculated in an inconsistent way.

In the function borrow() the variable debts is increased with a value excluding the fee.
However in repay() and liquidate() it is decreased with the same value as details[_id].debt is decreased,, which is including the fee.

This would mean that debts will end up in a negative value when all debts are repay-ed. Luckily the function repay() prevents this from happening.

In the mean time the value of debts isn't accurate.
This value is used directly or indirectly in: 
- utilizationRatio(), stabilityFee() calculateFeeIndex() of MochiProfileV0.sol 
- liveDebtIndex(), accrueDebt(), currentDebt() of MochiVault.sol

This means the entire debt and claimable calculations are slightly off.

## Proof of Concept
https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/vault/MochiVault.sol

function borrow(..)
    details[_id].debt = totalDebt; // includes the fee
    debts += _amount;     // excludes the fee 

function repay(..)
    debts -= _amount;  
    details[_id].debt -= _amount;

function liquidate(..)
   debts -= _usdm;
   details[_id].debt -= _usdm;

https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/vault/MochiVault.sol#L263-L268

https://github.com/code-423n4/2021-10-mochi/blob/806ebf2a364c01ff54d546b07d1bdb0e928f42c6/projects/mochi-core/contracts/profile/MochiProfileV0.sol#L272-L283

https://github.com/code-423n4/2021-10-mochi/blob/806ebf2a364c01ff54d546b07d1bdb0e928f42c6/projects/mochi-core/contracts/profile/MochiProfileV0.sol#L242-L256

https://github.com/code-423n4/2021-10-mochi/blob/806ebf2a364c01ff54d546b07d1bdb0e928f42c6/projects/mochi-core/contracts/profile/MochiProfileV0.sol#L258-L269

https://github.com/code-423n4/2021-10-mochi/blob/806ebf2a364c01ff54d546b07d1bdb0e928f42c6/projects/mochi-core/contracts/vault/MochiVault.sol#L66-L73

https://github.com/code-423n4/2021-10-mochi/blob/806ebf2a364c01ff54d546b07d1bdb0e928f42c6/projects/mochi-core/contracts/vault/MochiVault.sol#L79-L88

## Tools Used

## Recommended Mitigation Steps
In function borrow():
replace
    debts += _amount;
with
    debts += totalDebt

