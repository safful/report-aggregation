## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [MixinLockCore.sol has wrong comments](https://github.com/code-423n4/2021-11-unlock-findings/issues/122) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact

Wrong comment for withdraw(): modifier onlyLockManagerOrBeneficiary also allows the beneficiary to call this function and a beneficiary doesn't need to be a key manager/owner

[https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinLockCore.sol#L123](https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinLockCore.sol#L123)

Wrong comment for updateBeneficiary(): require statement also allows the beneficiary to call this function and a beneficiary doesn't need to be a key manager/owner

[https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinLockCore.sol#L189](https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinLockCore.sol#L189)

## Recommended Mitigation Steps
- Fix comments, because the implementation seems to be correct



