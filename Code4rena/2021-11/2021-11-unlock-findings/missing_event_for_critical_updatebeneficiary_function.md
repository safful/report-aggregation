## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing event for critical updateBeneficiary function](https://github.com/code-423n4/2021-11-unlock-findings/issues/75) 

# Handle

BouSalman


# Vulnerability details

## Vulnerability description 
The function **updateBeneficiary** in the **MixinLockCore** smart contract is used to lets the owner of the lock update the beneficiary account which receives funds on withdrawal. 

## Impact
Attackers can change the beneficiary address using this function before continue with the withdrawal function. Unlock protocol team and users can't log or monitor this critical changes.

## Proof of Concept
https://github.com/code-423n4/2021-11-unlock/blob/52f3f3d0524dda28aea327181c3479d85782007b/smart-contracts/contracts/mixins/MixinLockCore.sol#L192

## Tools Used
manual code review

## Recommended Mitigation Steps
define event and emit it to track changes done to the system.

